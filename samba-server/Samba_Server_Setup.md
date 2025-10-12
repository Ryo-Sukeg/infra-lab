# Samba Server Configuration (RHEL9.6)

## 概要
本構成は RHEL9.6 上に Samba サーバを構築し、Windows および Linux クライアントからアクセスできるファイル共有サーバを実現しています。同一ディレクトリは NFS サーバでも公開されていますが、本ファイルでは **Samba の設定** のみ記載しています。

---

## 環境構成
| 役割 | OS / ホスト名 | IPアドレス | 備考 |
|------|----------------|-------------|------|
| Sambaサーバ | RHEL9.6 | 192.168.56.103 | NFSサーバ兼用 |
| Linuxクライアント | Ubuntu 24.04 / Stream 9 / Alma 9.6 | 192.168.56.0/24 | |
| Windowsクライアント | Windows 10 / 11 | 192.168.56.0/24 | |

---

### 1. Samba インストール
```bash
sudo dnf install -y samba samba-client samba-common

### 2. 共有ディレクトリ作成
```
sudo mkdir -p /srv/samba/share
sudo chown -R nobody:nobody /srv/samba/share
sudo chmod -R 0777 /srv/samba/share
```
### 3. 設定ファイル編集  
/etc/samba/smb.conf の末尾に以下を追加：
```
[share]
    path = /srv/samba/share
    browseable = yes
    writable = yes
    valid users = sambauser
```
不要な共有（[homes], [printers] ）はコメントアウト。
### 4. Samba ユーザー設定  
OSユーザー作成：
```
sudo useradd sambauser
sudo passwd sambauser
```
Sambaユーザーとして登録：
```
sudo smbpasswd -a sambauser
sudo smbpasswd -e sambauser
```
登録済みユーザー確認：
```
sudo pdbedit -L
```
### 5. SELinux / Firewall 設定
```
sudo setsebool -P samba_export_all_rw on
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```
### 6. サービス起動
```
sudo systemctl enable --now smb nmb
sudo systemctl status smb nmb
```
### 7. クライアント接続確認
Linux クライアント
```
sudo mkdir -p /mnt/samba
sudo mount -t cifs //192.168.56.103/share /mnt/samba -o username=sambauser,password=SambaPass5566,vers=3.0
```
Windows クライアント
エクスプローラで入力：
\\192.168.56.103\share
### 8. 永続化設定（任意）
/etc/fstab に以下を追記：
# Sambaマウント設定
//192.168.56.103/share  /mnt/samba  cifs  credentials=/root/.smbcred,vers=3.0  0  0
.smbcred の例：
```
username=sambauser
password=SambaPass5566
```
### 9. NFSとの共存設定
既存 /srv/samba/share を /etc/exports にも設定して NFS 共有。
```
/srv/samba/share 192.168.56.0/24(rw,sync,no_root_squash)
sudo exportfs -r
```
