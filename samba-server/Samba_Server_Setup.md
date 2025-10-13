# Samba Server 構築手順（RHEL9.6）

本構成は RHEL9.6 上に Samba サーバを構築し、Windows および Linux クライアントからアクセスできるファイル共有サーバを実現しています。同一ディレクトリは NFS サーバでも公開されていますが、本ファイルでは **Samba の設定** のみ記載しています。  

## 構成概要
| 項 目 | 内 容 |
|------|------|
| Samba サーバ | RHEL 9.6（192.168.56.103）|
| サービス / Version | smbd ※1 , nmbd ※2 / Version 4.21.3 |
| 役 割 | ファイル共有サーバ（認証あり：share、認証なし：public） |
| Linux クライアント | CentOS Stream 9（192.168.56.101）/ Ubuntu 24.04.3（192.168.56.102）/ AlmaLinux 9.6（192.168.56.104）|
| Windows クライアント | Windows 11（172.100.21.x /24）|

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
# 認証あり

sudo mkdir -p /srv/samba/share
sudo chmod -R 0777 /srv/samba/share
sudo chown -R nobody:nobody /srv/samba/share
```
```
# 認証なし

sudo mkdir -p /srv/samba/public
sudo chmod -R 0777 /srv/samba/public
sudo chown -R nobody:nobody /srv/samba/public
sudo semanage fcontext -a -t samba_share_t "/srv/samba/public(/.*)?"
sudo restorecon -Rv /srv/samba/public
```
1-3. 設定ファイル編集  
/etc/samba/smb.conf の末尾に以下を追加：    ※ 不要な共有（ [homes], [printers] ）は # でコメントアウト
```
[share]
    path = /srv/samba/share
    browseable = yes
    writable = yes
    valid users = sambauser

[public]
      path = /srv/samba/public
      guest ok = yes
      writable = yes
      browsable = yes
```
1-4. Samba ユーザー設定
```
# OSユーザー作成

sudo useradd sambauser
sudo passwd sambauser
```
```
# Sambaユーザーとして登録

sudo smbpasswd -a sambauser		※ -a オプション : 新パスワードとともにローカルの smbpasswd ファイルに追加、既存時は変更して上書き
sudo smbpasswd -e sambauser		※ -e オプション : smbpasswd ファイル内の指定したユーザー名を有効にする
```
```
# Samba データベースに登録済みのユーザーを確認

sudo pdbedit -L		※ -Lオプション : 全ユーザアカウント一覧表示
```
1-5. SELinux / Firewall 設定  
```
sudo setsebool -P samba_export_all_rw on    ※ -P オプション : persistent の意味で再起動後も設定を維持
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```
1-6. サービス起動
```
sudo systemctl enable --now smb nmb
sudo systemctl status smb nmb
```
### 2. クライアント設定
2-1. パッケージインストール
```
sudo dnf install -y samba-client cifs-utils
```
2-2. クライアント接続確認 

Linux クライアント
```
# 認証ありフォルダ

sudo mkdir -p /mnt/samba
sudo mount -t cifs //192.168.56.103/share /mnt/samba -o username=sambauser,password=SambaPass123,vers=3.0
```
```
# 認証なしフォルダ

sudo mkdir -p /mnt/public
sudo mount -t cifs //192.168.56.103/public /mnt/public -o guest,vers=3.0
```
Windows クライアント（エクスプローラで入力）
```
\\192.168.56.103\share
```
2-3. 永続化設定（任意）  
/etc/fstab に以下を追記：    ※ 共有フォルダ自動マウント設定
```
# 認証ありフォルダ

//192.168.56.103/share  /mnt/samba  cifs  credentials=/root/.smbcred,vers=3.0  0  0
```
※ 上記の credentials で指定した `.smbcred` ファイルは root のみ読み書き可にして別途作成  
```
# .smbcred ファイル作成例

sudo vi /root/.smbcred

# 下記 username と password がファイル内容
---------------------------------
username=sambauser
password=SambaPass123
---------------------------------

sudo chmod 600 /root/.smbcred
```
```
# 認証なしフォルダ

//192.168.56.103/public  /mnt/public  cifs  guest,vers=3.0,_netdev  0  0
```
### 3. NFSとの共存設定
samba サーバの既存 /srv/samba/share を /etc/exports にも設定して NFS 共有
```
sudo /etc/exports

/srv/samba/share 192.168.56.0/24(rw,sync,root_squash)    ※ 追記

sudo exportfs -r
```
※ root_squash : NFS サーバーに（ローカルを除く）リモート接続の root ユーザーが root 権限を持つことを阻止し、ユーザー ID nfsnobody を割り当てる。無効は no_root_squash、全リモートユーザー 抑制は all_squash を使用
### 4. 備考
- CIFS : Microsoft が開発した SMB（Server Message Block）プロトコルを Windows 以外のシステムでも利用できるよう拡張したファイル共有プロトコル
- /etc/fstab : OS起動時に自動マウントするファイルシステム情報を記述するファイル  
記述順 : <ファイルシステム> <マウントポイント> <ファイルシステムの種類> <オプション> <ダンプフラグ> <fsck優先度>
- pdbedit プログラム : SAM データベース（Samba ユーザーのデータベース）内に保持されるユーザーアカウントを管理するために利用される
- pdbedit : SAM データベースを管理。ユーザーアカウント追加・削除・変更・一覧表示・取り込み
- smbpasswd : ユーザーの SMB パスワードを変更する
- mount コマンドを使用し共有ファイルに接続確認後再起動、履歴から前回の mount コマンドを実行して共有フォルダに移動してもファイルが表示されない事象あり、再起動後に再度同じことを試して表示されることがありました（fstab に記載後は問題なし）

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
Linux クライアントからの samba サーバ接続確認（mount せずに使用可）
```
# 共有一覧表示
smbclient -L //192.168.56.103 -U sambauser

# 接続
smbclient //192.168.56.103/share -U sambauser
```
