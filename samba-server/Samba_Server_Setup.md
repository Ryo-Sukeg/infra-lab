# Samba Server 構築手順 ( RHEL9.6 )

本構成は RHEL9.6 上に Samba サーバを構築し、Windows および Linux クライアントからアクセスできるファイル共有サーバを実現しています。同一ディレクトリは NFS サーバでも公開されていますが、本ファイルでは **Samba の設定** のみ記載しています。  

## 構成概要
| 項 目 | 内 容 |
|------|------|
| Samba サーバ | RHEL 9.6 ( 192.168.56.103 ) |
| サービス / Version | smbd ※1 , nmbd ※2 / Version 4.21.3 |
| 役割 | ファイル共有サーバ |
| Linux クライアント | CentOS Stream 9 ( 192.168.56.101 ) / Ubuntu 24.04.3 ( 192.168.56.102 ) / AlmaLinux 9.6 ( 192.168.56.104 ) |
| Windows クライアント | Windows 11 ( 172.100.21.x /24 ) |

※1 smbd : Windowsネットワークのクライアントに対しファイル共有やプリンタ共有のサービスを提供  
※2 nmbd : NetBIOSの名前解決を行うサービス、IPアドレスに対応するコンピュータ名を解決し他マシンを発見できるようにする

---
## 手順
### 1. Samba サーバ設定
1-1. Samba インストール
```
sudo dnf install -y samba samba-client samba-common
```
1-2. 共有ディレクトリ作成
```
sudo mkdir -p /srv/samba/share
sudo chown -R nobody:nobody /srv/samba/share
sudo chmod -R 0777 /srv/samba/share
```
1-3. 設定ファイル編集  
/etc/samba/smb.conf の末尾に以下を追加：    ※ 不要な共有 ( [homes], [printers] ) は # でコメントアウト
```
[share]
    path = /srv/samba/share
    browseable = yes
    writable = yes
    valid users = sambauser
```
1-4. Samba ユーザー設定  
OSユーザー作成：
```
sudo useradd sambauser
sudo passwd sambauser
```
Sambaユーザーとして登録：
```
sudo smbpasswd -a sambauser		※ -a オプション : 新パスワードとともにローカルの smbpasswd ファイルに追加、既存時は変更して上書き
sudo smbpasswd -e sambauser		※ -e オプション : smbpasswd ファイル内の指定したユーザー名を有効にする
```
Samba データベースに登録済みのユーザー確認：
```
sudo pdbedit -L		※ -Lオプション : 全ユーザアカウント一覧表示
```
1-5. SELinux / Firewall 設定
```
sudo setsebool -P samba_export_all_rw on
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```
1-6. サービス起動
```
sudo systemctl enable --now smb nmb
sudo systemctl status smb nmb
```
### 2. クライアント設定
2-1. クライアント接続確認
Linux クライアント
```
sudo mkdir -p /mnt/samba
sudo mount -t cifs //192.168.56.103/share /mnt/samba -o username=sambauser,password=SambaPass123,vers=3.0
```
Windows クライアント
エクスプローラで入力：
```
\\192.168.56.103\share
```
2-2. 永続化設定（任意）
/etc/fstab に以下を追記：
Sambaマウント設定
```
//192.168.56.103/share  /mnt/samba  cifs  credentials=/root/.smbcred,vers=3.0  0  0
```
※ 上記の credentials で指定した `.smbcred` ファイルは rootのみ読み書き可にして別途作成
.smbcredファイル作成例：
```
sudo vi /root/.smbcred

# 以下ファイル内容
---------------------------------
username=sambauser
password=SambaPass123
---------------------------------

sudo chmod 600 /root/.smbcred
```
### 3. NFSとの共存設定
既存 /srv/samba/share を /etc/exports にも設定して NFS 共有
```
/srv/samba/share 192.168.56.0/24(rw,sync,no_root_squash)
sudo exportfs -r
```
### 4. 備考
- CIFS : Microsoft が開発した SMB ( Server Message Block ) プロトコルを Windows 以外のシステムでも利用できるように拡張したファイル共有プロトコル
- /etc/fstab : OS起動時に自動マウントするファイルシステム情報を記述するファイル  
記述順 : <ファイルシステム> <マウントポイント> <ファイルシステムの種類> <オプション> <ダンプフラグ> <fsck優先度>
- pdbedit プログラム : SAM データベース (Samba ユーザーのデータベース) 内に保持されるユーザーアカウントを管理するために利用され root だけが実行できる
- pdbedit : SAM データベースを管理。ユーザーアカウント追加・削除・変更・一覧表示・取り込み
- smbpasswd : ユーザーの SMB パスワードを変更する

### 気になったコマンド
samba バージョン確認
```
smbd -V
samba --version
```
ブート時のマウント失敗ログ確認
```
journalctl -b | grep mount
dmesg | grep CIFS
```
関連エラーメッセージ
```
journalctl -b
```
