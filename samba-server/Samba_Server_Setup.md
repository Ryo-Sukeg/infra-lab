# Samba Server 構築手順（RHEL9.6）

本構成は RHEL9.6 上に Samba サーバを構築し、Windows と Linux クライアントからアクセスできるファイル共有サーバを実現しています。  

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
sudo chmod -R 2770 /srv/samba/share
sudo chown -R root:smbusers /srv/samba/share
```
※ 2770 について：2 → SetGID（ディレクトリの場合：ディレクトリ内に作成された新ファイル・ディレクトリが親ディレクトリのグループ所有者を自動継承）770 → 所有者とグループのみ読み書き可
```
# 認証なし

sudo mkdir -p /srv/samba/public
sudo chmod -R 0777 /srv/samba/public
sudo chown -R nobody:nobody /srv/samba/public
sudo semanage fcontext -a -t samba_share_t "/srv/samba/public(/.*)?"
sudo restorecon -Rv /srv/samba/public
```
※ `semanage fcontext` と `restorecon` は 4. 備考 の下の SELinux の設定についてに記載  
他ユーザーが作成したファイルを削除・変更不可にするには、対象ディレクトリに sticky ビットを追加する（例：chmod -R 1770 ディレクトリパス、chmod o+t ディレクトリパス）

1-3. 設定ファイル編集  
/etc/samba/smb.conf の末尾に以下を追加：    ※ 不要な共有（ [homes], [printers] ）は # でコメントアウト
```
[global]
   workgroup = WORKGROUP
   security = user
   map to guest = Bad User
   guest account = nobody

[share]
   path = /srv/samba/share
   browseable = yes
   writable = yes
   valid users = @smbusers
   create mask = 0660
   directory mask = 0770

[public]
   path = /srv/samba/public
   browseable = yes
   writable = yes
   guest ok = yes
   force user = nobody
   force group = nobody
   create mask = 0666
   directory mask = 0777
```
- workgroup = WORKGROUP：Samba サーバの Windows ネットワーク上の所属グループ名。デフォルト
- map to guest = Bad User：存在しないユーザーを guest 扱いに変換
- guest account = nobody：guest ユーザーをシステムユーザー nobody に紐づけ
- guest ok = yes：該当共有で guest 接続を許可
- force user/group = nobody：ファイル作成時の所有者を固定

1-4. Samba ユーザー作成と登録  

Linux ユーザー作成
```
sudo useradd -M -s /sbin/nologin alma
sudo passwd alma
```
- -M：ホームディレクトリを作成しない
- -s：使用するシェルを指定
- /sbin/nologin：システムログイン不可を意味する特殊シェル（システムログイン不要の場合）  

Samba 認証ユーザー登録
```
sudo smbpasswd -a alma		※ -a オプション : 新パスワードと一緒にローカルの smbpasswd ファイルに追記・既存時は上書きされる
sudo smbpasswd -e alma		※ -e オプション : smbpasswd ファイル内の指定したユーザー名を有効にする
```
1-5. ユーザー／グループ単位のアクセス制御設定  

グループ作成とメンバー登録
```
sudo groupadd -g 2001 smbusers
sudo usermod -aG smbusers alma
```
Samba データベースに登録済みのユーザーを確認
```
sudo pdbedit -L		※ -Lオプション : 全ユーザアカウント一覧表示
```
1-6. SELinux / Firewall 設定  
```
sudo setsebool -P samba_export_all_rw on    ※ -P オプション : persistent の意味で再起動後も設定を維持
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd --reload
```
1-7. サービス起動
```
sudo systemctl enable --now smb nmb
sudo systemctl status smb nmb
```
### 2. クライアント設定
2-1. Linux クライアント・パッケージインストール
```
sudo dnf install -y samba-client cifs-utils
```
2-2. Linux クライアント・マウント設定 
```
# 認証ありフォルダ

sudo mkdir -p /mnt/samba/share
sudo mount -t cifs //192.168.56.103/share /mnt/samba/share \
    -o username=alma,password=sambapass123,vers=3.0,gid=2001,file_mode=0660,dir_mode=0770
```
```
# 認証なしフォルダ

sudo mkdir -p /mnt/samba/public
sudo mount -t cifs //192.168.56.103/public /mnt/samba/public \
    -o guest,vers=3.0,uid=65534,gid=65534,file_mode=0666,dir_mode=0777
```
2-3. Windows クライアント接続確認（エクスプローラで入力）
```
# 認証あり
\\192.168.56.103\share

# 認証なし
\\192.168.56.103\public
```
2-4. Linux クライアント・永続化設定（任意）  
/etc/fstab に以下を追記：    ※ 共有フォルダ自動マウント設定
```
//192.168.56.103/public  /mnt/samba/public  cifs  guest,uid=65534,gid=65534,file_mode=0666,dir_mode=0777 ,vers=3.0,_netdev  0  0
//192.168.56.103/share /mnt/samba/share cifs credentials=/root/.smbcred,gid=2001,file_mode=0660,dir_mode=0770,vers=3.0,_netdev 0 0
```
※ credentials で指定した `.smbcred` ファイルは root のみ読み書き可にして別途作成。↑ uid/gid=65534 は nobody  
```
# .smbcred ファイル作成例

sudo vi /root/.smbcred

# 下記 username と password がファイル内容
---------------------------------
username=alma
password=sambapass123
---------------------------------

sudo chmod 600 /root/.smbcred
```
### 3. NFSとの共存設定
samba サーバの既存 /srv/samba/share を /etc/exports にも設定して NFS 共有（※ 今回は設定していません）
```
sudo vi /etc/exports

/srv/samba/share 192.168.56.0/24(rw,sync,root_squash)    ※ exports ファイル内に追記

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

### SELinux の設定について
コンテキスト：SELinux を設定するための情報、プロセスやファイルに付与されるセキュリティラベルの情報で `ユーザー：ロール：タイプ：機密ラベル` という4つの要素で構成される（ls -Z で確認可）  
semanage fcontext：永続的に SELinux コンテキストを変更する    
-a オプション：新しいレコードを追加  
-t オプション：タイプ指定  
-l オプション：現在設定されているコンテキスト一覧  
-d オプション：コンテキストのルール削除、ルールで使われている正規表現を指定  
restorecon：変更した設定をファイルに反映させる

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
全共有ディレクトリを読み書き可能でエクスポート
```
samba_export_all_rw --> on
```
samba がポートマッパーを使用する設定、ポートマッパーは主に NFS などの他サービスで使用されるため NFS 共有と連携する場合に有効にする
```
samba_portmapper --> on
```
samba を NFS 共有として利用できるようにする、samba 共有を NFS としてエクスポートできる
```
samba_share_nfs --> on
```
