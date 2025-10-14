# Samba Server 検証結果記録

## 構成概要

| サーバ種別 | O S | FQDN | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| Linux client | CentOS Stream 9 | stream.lab.lam | 192.168.56.101 | 共有フォルダにtxtアップロード |
| Linux client | Ubuntu 24.04.3 | ubuntu.lab.lam | 192.168.56.102 | 共有フォルダにtxtアップロード |
| **samba** / NFS | RHEL 9.6 | **rhel.lab.lam** | **192.168.56.103** | **ファイル共有サーバ** |
| Linux client | AlmaLinux 9.6 | alma.lab.lam | 192.168.56.104 | 検証テスト |
| Windows client | Windows 11 | win11-test | 172.21.100.x/24 | 検証テスト |

---

### 1. Samba サーバの状態確認

```
sudo systemctl status smb nmb
```
出力結果：
```
$ sudo systemctl status smb nmb

● smb.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smb.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-10-14 12:00:37 JST; 14min ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 914 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 11066)
     Memory: 13.7M
        CPU: 371ms
     CGroup: /system.slice/smb.service
             tq 914 /usr/sbin/smbd --foreground --no-process-group
             tq 962 /usr/sbin/smbd --foreground --no-process-group
             tq 965 /usr/sbin/smbd --foreground --no-process-group
             mq1336 /usr/sbin/smbd --foreground --no-process-group

10月 14 12:00:37 RHEL9.6 systemd[1]: Starting Samba SMB Daemon...
10月 14 12:00:37 RHEL9.6 smbd[914]: [2025/10/14 12:00:37.667644,  0] ../../source3/smbd/server.c:1965(main)
10月 14 12:00:37 RHEL9.6 smbd[914]:   smbd version 4.21.3 started.
10月 14 12:00:37 RHEL9.6 smbd[914]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
10月 14 12:00:37 RHEL9.6 systemd[1]: Started Samba SMB Daemon.

● nmb.service - Samba NMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/nmb.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-10-14 12:00:37 JST; 14min ago
       Docs: man:nmbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 857 (nmbd)
     Status: "nmbd: ready to serve connections..."
      Tasks: 1 (limit: 11066)
     Memory: 14.6M
        CPU: 496ms
     CGroup: /system.slice/nmb.service
             mq857 /usr/sbin/nmbd --foreground --no-process-group

10月 14 12:00:36 RHEL9.6 systemd[1]: Starting Samba NMB Daemon...
10月 14 12:00:37 RHEL9.6 nmbd[857]: [2025/10/14 12:00:37.276758,  0] ../../source3/nmbd/nmbd.c:901(main)
10月 14 12:00:37 RHEL9.6 nmbd[857]:   nmbd version 4.21.3 started.
10月 14 12:00:37 RHEL9.6 nmbd[857]:   Copyright Andrew Tridgell and the Samba Team 1992-2024
10月 14 12:00:37 RHEL9.6 systemd[1]: Started Samba NMB Daemon.
```

### 2. クライアントからのアクセス確認

2-1. Linuxクライアントでの検証 (AlmaLinux 9.6)

共有フォルダのマウント
```bash
# 公開共有ディレクトリ (guest OK)  ※ マウント元サーバをFQDNで指定する場合はDNSサーバ起動必要、IPアドレス指定でも可
sudo mount -t cifs //rhel.lab.lan/public /mnt/public -o guest,vers=3.0

# 認証付き共有ディレクトリ
sudo mount -t cifs //rhel.lab.lan/share /mnt/samba -o username=sambauser,password=SambaPass123,vers=3.0
```
マウント確認
```
df -hT | grep cifs
```
出力結果：
```
$ df -hT | grep cifs
//rhel.lab.lan/public           cifs        17G  2.4G   15G   14% /mnt/public
//rhel.lab.lan/share            cifs        17G  2.4G   15G   14% /mnt/samba
```
読み書きテスト
```
sudo touch /mnt/samba/alma_test.txt
ls -l /mnt/samba/alma_test.txt
```
出力結果：
```
$ sudo touch /mnt/samba/alma_test.txt
$ ls -l /mnt/samba/alma_test.txt
-rwxr-xr-x. 1 root root 0 10月 14 14:04 /mnt/samba/alma_test.txt
```
2-2. Windowsクライアントでの検証（Windows 11）

ネットワークアクセス確認　※ Windows のエクスプローラに入力して確認
```
# 認証あり共有フォルダ  
エクスプローラ：\\192.168.56.103\share  
ユーザ名：sambauser  
パスワード：********

# 認証なし共有フォルダ  
エクスプローラ：\\192.168.56.103\public
```
アクセス後、共有フォルダにファイル作成・削除が可能であることを確認  
※ LinuxDNS の FQDN 反映方法は本ページ下側に記載

### 3. SELinux / Firewall 動作確認  

3-1. SELinuxブール値確認
```
sudo getsebool -a | grep samba
samba_enable_home_dirs --> off
samba_export_all_rw --> on
samba_export_all_ro --> off
```
3-2. ファイアウォール設定
```
sudo firewall-cmd --list-all | grep services
services: dhcpv6-client mdns samba ssh
```
3-3. 確認結果
- SELinux有効状態で書き込み可
- UDP/TCP 445, 139 ポート開放済み
- DNS名アクセス正常（NetBIOS不要）

### 4. 検証まとめ
| 項 目 | 結果 |
|------|------|
| Linuxクライアントからの接続	| 正常（認証 / guest共にOK）|
| Windowsクライアントからの接続	| 正常（エクスプローラ経由で確認）|
| SELinux有効状態でのアクセス	| 書き込み可能 |
| ファイアウォール設定 | 正常（sambaサービス有効）|
| DNS解決・名前解決 | 正常（lab.lan ドメイン動作）|
