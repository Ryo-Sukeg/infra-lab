# NFS Server 検証結果記録

## 構成概要

| サーバ種別 | O S | FQDN | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| Linux client | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | 共有フォルダにtxtアップロード |
| Linux client | Ubuntu 24.04.3 | ubuntu.lab.lan | 192.168.56.102 | 共有フォルダにtxtアップロード |
| samba / **NFS** | RHEL 9.6 | **rhel.lab.lan** | **192.168.56.103** | **ファイル共有サーバ** |
| Linux client | AlmaLinux 9.6 | alma.lab.lan | 192.168.56.104 | 共有フォルダにtxtアップロード  |

---

### 1. NFS サーバの各種状態確認  
1-1. NFS サーバ起動状態確認
```
systemctl status nfs-server
```
出力結果：
```
$ systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             mqorder-with-mounts.conf
     Active: active (exited) since Sun 2025-11-02 16:01:14 JST; 29min ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 875 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 877 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 935 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload >
   Main PID: 935 (code=exited, status=0/SUCCESS)
        CPU: 145ms

11月 02 16:01:13 RHEL9.6 systemd[1]: Starting NFS server and services...
11月 02 16:01:14 RHEL9.6 systemd[1]: Finished NFS server and services.
```
1-2. エクスポートディレクトリパーミッション確認
```
ls -l /srv/nfs
```
出力結果：
```
$ ls -l /srv/nfs
合計 0
drwxr-xr-x. 2 nobody nobody   37 10月 29 01:28 public
drwxrws---. 2 root   devgroup 73 10月 30 01:07 share
drwx------. 2 root   root      6 11月  1 02:30 system
```
1-3. エクスポート設定確認  
```
cat /etc/exports
```
出力結果：
```
$ cat /etc/exports
/srv/nfs/public 192.168.56.0/24(ro,sync,root_squash)
/srv/nfs/share 192.168.56.0/24(rw,sync,root_squash)
/srv/nfs/system 192.168.56.103(rw,sync,root_squash)
```
1-4. 有効エクスポート設定全表示
```
sudo exportfs -v
```
出力結果：
```
$ sudo exportfs -v
/srv/nfs/system
	192.168.56.103(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/srv/nfs/public
	192.168.56.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)
/srv/nfs/share
	192.168.56.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
- wdelay：書き込み遅延調整、パフォーマンス最適化。デフォルト
- hide：サブエクスポート非表示。デフォルト
- no_subtree_check：パフォーマンス優先、転送高速化。デフォルト
- sec=sys：UID/GIDベース標準UNIX認証使用。デフォルト
- secure：クライアント接続を特権ポート（1024番未満）のみ許可。デフォルト
- no_all_squash：全ユーザを匿名ユーザに置き換えしない。root_squash では root だけ置き換え、一般ユーザは UID/GID のまま。※ root_squash と no_all_squash はデフォルト

### 2. クライアントからのアクセス確認

2-1. クライアントのマウント状態確認
```
showmount -e 192.168.56.103
```
出力結果：
```
$ showmount -e 192.168.56.103
Export list for 192.168.56.103:
/srv/nfs/share  192.168.56.0/24
/srv/nfs/public 192.168.56.0/24
/srv/nfs/system 192.168.56.103
```
df -hT | grep nfs
```
出力結果：
```
$ df -hT | grep nfs
192.168.56.103:/srv/nfs/share  nfs4        17G  2.4G   15G   14% /mnt/nfs/share
192.168.56.103:/srv/nfs/public nfs4        17G  2.4G   15G   14% /mnt/nfs/public
```
2-2. 読み書きテスト  

public（読み取り専用）
```
# 一般ユーザのファイル保存
$ touch /mnt/nfs/public/test_stream.txt
touch: '/mnt/nfs/public/test_stream.txt' に touch できません: 読み込み専用ファイルシステムです

# 一般ユーザのファイル読み取り
[ryo.s@Stream9 ~]$ cd /mnt/nfs/public
[ryo.s@Stream9 public]$ ll
合計 4
-rwxr-xr-x. 1 root root 27 10月 29 01:24 NFS_Server_readonly.txt
[ryo.s@Stream9 public]$ cat NFS_Server_readonly.txt
NFS Server test from RHEL.
```
share（devgroup 限定）
```
# 一般ユーザのファイル保存
$ touch /mnt/nfs/share/test_stream.txt
touch: '/mnt/nfs/share/test_stream.txt' に touch できません: 許可がありません

# 登録ユーザのファイル保存
[root@Stream9 ~]# su - stream
最終ログイン: 2025/11/02 (日) 18:23:29 JST 日時 pts/0
[stream@Stream9 ~]$ touch /mnt/nfs/share/test_stream.txt

# NFSサーバ /srv/nfs/share ディレクトリ
[root@RHEL9 share]# ll
合計 12
-rw-rw----. 1 alma   devgroup 95 10月 29 23:45 alma_test.txt
-rw-rw----. 1 stream devgroup 96 10月 30 01:07 stream_test.txt
-rw-rw----. 1 stream devgroup  0 11月  2 18:24 test_stream.txt
-rw-rw----. 1 ubuntu devgroup 96 10月 29 23:41 ubuntu_test.txt
```
```
system（管理者専用）
```
# クライアントからファイル送信
[root@Stream9 ~]# su - ryo.s
最終ログイン: 2025/11/02 (日) 18:38:04 JST 日時 pts/0
[ryo.s@Stream9 ~]$ touch test_from_stream.txt
[ryo.s@Stream9 ~]$ scp ./test_from_stream.txt root@192.168.56.103:/srv/nfs/system
root@192.168.56.103's password:
test_from_stream.txt                                     100%    0     0.0KB/s   00:00

# NFS サーバ側の system ディレクトリアクセス確認と受信ファイル確認
[ryo.s@RHEL9 ~]$ cd /srv/nfs/system
-bash: cd: /srv/nfs/system: 許可がありません
[ryo.s@RHEL9 ~]$ su -
パスワード:
最終ログイン: 2025/11/02 (日) 18:25:44 JST 日時 pts/0
[root@RHEL9 ~]# cd /srv/nfs/system
[root@RHEL9 system]# ll
合計 0
-rw-r--r--. 1 root root 0 11月  2 19:17 test_from_stream.txt
```

###3. SELinux / Firewall 動作確認（NFS Server）  
3.1 SELinux ブール値確認
```
getsebool -a | grep nfs | grep ' on'
```
出力結果：
```
$ getsebool -a | grep nfs | grep ' on'
nfs_export_all_ro --> on
nfs_export_all_rw --> on
```
3.2 Firewall 設定確認
sudo firewall-cmd --list-all | grep services
```
出力結果：
```
$ sudo firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs ntp rpc-bind samba ssh
```

### 検証結果まとめ
| 項 目 | 結 果 |
|------|------|
| クライアントからの接続 | 正常（public・share：共に問題なし）|
| SELinux有効状態でのアクセス | public・share・system：問題なし |
| ファイアウォール設定 | 正常（nfsサービス有効）|

