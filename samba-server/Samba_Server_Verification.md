# Samba Server 検証結果記録

## 構成概要

| サーバ種別 | O S | FQDN | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| Linux client | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | txtアップロード |
| Linux client | Ubuntu 24.04.3 | ubuntu.lab.lan | 192.168.56.102 | txtアップロード |
| **samba** / NFS | RHEL 9.6 | **rhel.lab.lan** | **192.168.56.103** | **Windows / Linux ファイル共有サーバ** |
| Linux client | AlmaLinux 9.6 | alma.lab.lan | 192.168.56.104 | 検証テスト |
| Windows client | Windows 11 |   - | 172.21.100.x/24 | 検証テスト |

---

### 1. Samba サーバ状態確認

```
systemctl status smb nmb
```
出力結果：
```
$ systemctl status smb nmb

● smb.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smb.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-11-04 16:36:38 JST; 9h ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 2006 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 7 (limit: 11066)
     Memory: 13.1M
        CPU: 18.440s
     CGroup: /system.slice/smb.service
             tq2006 /usr/sbin/smbd --foreground --no-process-group
             tq2008 /usr/sbin/smbd --foreground --no-process-group
             tq2009 /usr/sbin/smbd --foreground --no-process-group
             tq2786 /usr/sbin/smbd --foreground --no-process-group
             tq3191 /usr/sbin/smbd --foreground --no-process-group
             tq3192 /usr/sbin/smbd --foreground --no-process-group
             mq3550 /usr/sbin/smbd --foreground --no-process-group

11月 05 01:26:46 RHEL9.6 samba-dcerpcd[3495]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
11月 05 01:26:47 RHEL9.6 rpcd_classic[3505]: [2025/11/05 01:26:47.022595,  0] ../../source3/rpc_server/rpc_worker.c:1148(rpc_worker_main)
11月 05 01:26:47 RHEL9.6 rpcd_classic[3505]:   rpcd_classic version 4.21.3 started.
11月 05 01:26:47 RHEL9.6 rpcd_classic[3505]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3507]: [2025/11/05 01:26:47.190991,  0] ../../source3/rpc_server/rpc_worker.c:1148(rpc_worker_main)
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3507]:   rpcd_winreg version 4.21.3 started.
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3507]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3509]: [2025/11/05 01:26:47.443135,  0] ../../source3/rpc_server/rpc_worker.c:1148(rpc_worker_main)
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3509]:   rpcd_winreg version 4.21.3 started.
11月 05 01:26:47 RHEL9.6 rpcd_winreg[3509]:   Copyright Andrew Tridgell and the Samba Team 1992-2024

● nmb.service - Samba NMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/nmb.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-11-04 16:36:38 JST; 9h ago
       Docs: man:nmbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 2004 (nmbd)
     Status: "nmbd: ready to serve connections..."
      Tasks: 1 (limit: 11066)
     Memory: 2.8M
        CPU: 3.984s
     CGroup: /system.slice/nmb.service
             mq2004 /usr/sbin/nmbd --foreground --no-process-group

11月 04 16:36:38 RHEL9.6 systemd[1]: Starting Samba NMB Daemon...
11月 04 16:36:38 RHEL9.6 nmbd[2004]: [2025/11/04 16:36:38.579518,  0] ../../source3/nmbd/nmbd.c:901(main)
11月 04 16:36:38 RHEL9.6 nmbd[2004]:   nmbd version 4.21.3 started.
11月 04 16:36:38 RHEL9.6 nmbd[2004]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
11月 04 16:36:38 RHEL9.6 systemd[1]: Started Samba NMB Daemon.
```
### 2. クライアントからのアクセス確認

2-1. Linux クライアント (AlmaLinux)

共有フォルダのマウント
```
# 公開共有 ※ FQDN 指定時は DNS サーバ要起動、IPアドレス指定でも可
sudo mount -t cifs //rhel.lab.lan/public /mnt/samba/public  -o guest,vers=3.0,uid=65534,gid=65534,file_mode=0666,dir_mode=0777

# 認証付共有
sudo mount -t cifs //rhel.lab.lan/share /mnt/samba/share  -o username=ryo.s,password=ryotepass5566,vers=3.0,gid=2001,file_mode=0660,dir_mode=0770
```
マウント確認
```
df -hT | grep cifs
```
出力結果：
```
$ df -hT | grep cifs
//rhel.lab.lan/public           cifs        17G  2.4G   15G   14% /mnt/samba/public
//rhel.lab.lan/share            cifs        17G  2.4G   15G   14% /mnt/samba/share
```
読み書きテスト  
```
# 公開共有（public）

$ vi /mnt/samba/public/alma_test.txt
$ ls -l /mnt/samba/public/alma_test.txt
-rw-rw-rw-. 1 nobody nobody  0 11月  5 02:36 alma_test.txt
```
```
# 認証付共有（share）

[root@Alma9 ~]# su - alma
最終ログイン: 2025/11/04 (火) 16:10:40 JST 日時 pts/0
[alma@Alma9 ~]$ vi /mnt/samba/share/alma_test.txt
[alma@Alma9 ~]$ ls -l /mnt/samba/share/alma_test.txt
-rw-rw----. 1 root smbusers 29 11月  5 02:57 /mnt/samba/share/alma_test.txt
```
2-2. Windows クライアント

ネットワークアクセス確認　※ Windows エクスプローラから確認
```
# 認証ありフォルダ

エクスプローラ：\\rhel.lab.lan\share  
ユーザ名：alma  
パスワード：********

# 認証なしフォルダ

エクスプローラ：\\rhel.lab.lan\public
```
アクセス後、共有フォルダにファイル作成・削除可能であることを確認（README.md 下方に画像添付）  
※ Linux DNS の FQDN 反映方法は `DNS Server 構築手順記録` 下側 `Windows で Linux DNSサーバの FQDN を反映させる` に記載

### 3. SELinux / Firewall 動作確認（Samba Server）  

3-1. SELinux ブール値確認
```
sudo getsebool -a | grep samba
```
出力結果：
```
$ sudo getsebool -a | grep samba
samba_create_home_dirs --> off
samba_domain_controller --> off
samba_enable_home_dirs --> on
samba_export_all_ro --> off
samba_export_all_rw --> on
samba_load_libgfapi --> off
samba_portmapper --> off
samba_run_unconfined --> off
samba_share_fusefs --> off
samba_share_nfs --> off
sanlock_use_samba --> off
tmpreaper_use_samba --> off
use_samba_home_dirs --> off
virt_use_samba --> off
```
3-2. Firewall 設定確認
```
sudo firewall-cmd --list-all | grep services
```
出力結果：
```
$ sudo firewall-cmd --list-all | grep service
services: cockpit dhcpv6-client mountd nfs ntp rpc-bind samba ssh
```
### 検証結果まとめ
| 項 目 | 結 果 |
|------|------|
| Linuxクライアントからの接続	| 正常（認証あり・認証なし：共に問題なし）|
| Windowsクライアントからの接続	| 正常（認証あり・認証なし：共に問題なし）|
| SELinux有効状態でのアクセス	| 認証あり・認証なし：共に書き込み可 |
| ファイアウォール設定 | 正常（sambaサービス有効）|
| DNS解決・名前解決 | 正常（lab.lan ドメイン動作）|
