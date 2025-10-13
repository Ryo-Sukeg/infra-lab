# Samba Server 検証結果記録

## 構成概要

| サーバ種別 | O S | ホスト名 | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| Linux client | CentOS Stream 9 | stream9.6 | 192.168.56.101 | 共有フォルダにtxtアップロード |
| Linux client | Ubuntu 24.04.3 | ubuntu24 | 192.168.56.102 | 共有フォルダにtxtアップロード |
| **samba** / NFS | **RHEL 9.6** | **rhel9.6** | **192.168.56.103** | **ファイル共有サーバ** |
| Linux client | AlmaLinux 9.6 | alma9.6 | 192.168.56.104 | 検証テスト |
| Windows client | Windows 11 | win11-test | 172.21.100.x/24 | 検証テスト |

---

### 1. Samba サーバ状態確認


### 2. クライアントからのアクセス確認

2-1. Linuxクライアントでの検証 (AlmaLinux 9.6)

共有フォルダのマウント
```bash
# 公開共有ディレクトリ (guest OK)
sudo mount -t cifs //file.lab.lan/public /mnt/public -o guest

# 認証付き共有ディレクトリ
sudo mount -t cifs //file.lab.lan/share /mnt/share -o username=sambauser
```
マウント確認
```
df -hT | grep cifs
//file.lab.lan/public  cifs  100G  1.5G  98G   2% /mnt/public
//file.lab.lan/share   cifs  100G  1.5G  98G   2% /mnt/share
```
読み書きテスト
```
touch /mnt/share/testfile.txt
ls -l /mnt/share/testfile.txt
-rw-r--r--. 1 sambauser sambauser 0 Oct 07 22:00 /mnt/share/testfile.txt
```
2-2. Windowsクライアントでの検証（Windows 11）

ネットワークアクセス確認

認証付き共有アクセス
エクスプローラ > \\file.lab.lan\share
ユーザ名: sambauser
パスワード: ********

アクセス後、共有フォルダにファイル作成・削除が可能であることを確認

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
