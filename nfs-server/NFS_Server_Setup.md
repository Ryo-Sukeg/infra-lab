# NFS Server 構築手順（RHEL9.6）  

## 構成概要
| 項 目 | 内 容 |
|------|------|
| NFS サーバ | RHEL 9.6（192.168.56.103）|
| サービス / Version | nfsd / NFSv4 |
| 役 割 | ファイル共有サーバ（公開共有：public、特定グループ共有：share、管理サーバ専用：system）|
| クライアント | CentOS Stream 9（192.168.56.101）/ Ubuntu 24.04.3（192.168.56.102）/ AlmaLinux 9.6（192.168.56.104）|

---
## 手順
### 1. NFS サーバ設定  
1-1. NFS インストール
```
sudo dnf install -y nfs-utils
```
1-2. 共有ディレクトリ作成
```
# 公開共有（閲覧専用）

sudo mkdir -p /srv/nfs/public
sudo chmod -R 755 /srv/nfs/public
sudo chown -R nobody:nobody /srv/nfs/public
```
```
# 特定グループ共有（特定 UID/GID のみ書込可） ※ 共有用ユーザ作成方法は 3. 、共有内ファイル他ユーザー編集不可時対処法は 4.に記載

sudo mkdir -p /srv/nfs/share
sudo chmod -R 2770 /srv/nfs/share　※ 2770 について：2 → SetGID（グループ継承）770 → 所有者とグループのみ読み書き可
sudo chown -R root:devgroup /srv/nfs/share
```
```
# 管理者専用（管理者のみ書込可、バックアップ格納用）

sudo mkdir -p /srv/nfs/system
sudo chmod -R 700 /srv/nfs/system
sudo chown -R root:root /srv/nfs/system
```
1-3. 設定ファイル編集  
/etc/exports ファイルに以下を追加：
```
/srv/nfs/public 192.168.56.0/24(ro,sync,root_squash)
/srv/nfs/share 192.168.56.0/24(rw,sync,root_squash)
/srv/nfs/system 192.168.56.103(rw,sync,root_squash)
```
↑ NFS サーバで公開するディレクトリ情報を記載。左から "共有ディレクトリのパス　アクセス可能なネットワーク(マウントオプション)"  
- ro：読み取り専用。デフォルト
- rw：読み書き専用  
- sync：書き込み時に同期、データを確実にディスクへ書き込んでから応答。デフォルト  
- root_squash：クライアント root を匿名ユーザ nobody に置き換え（権限昇格防止）。デフォルト

1-4. 設定反映と確認  
エクスポート反映・確認（一覧）
```
sudo exportfs -r
sudo exportfs -v
```
1-5. サービス起動設定・状態確認
```
sudo systemctl enable --now nfs-server
sudo systemctl status nfs-server
```
必要に応じて rpcbind や nfs-mountd の状態も確認：　※ rpcbind、nfs-mountd については備考に記載
```
sudo systemctl status rpcbind
sudo systemctl status nfs-mountd
```
※ 問題があれば journalctl -u nfs-server -e でログ確認

1-6. ファイアウォール設定（firewalld 使用時）  
NFS は複数サービス（nfs, mountd, rpc-bind）使用
```
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload

# 現在の設定確認
sudo firewall-cmd --list-all
```
1-7. SELinux 設定  
有効でも読み書きできるようブール値を設定
```
# NFS書き込み許可（NFS共有に対する一般的設定）
sudo setsebool -P nfs_export_all_rw on

# Samba と同一ディレクトリを使う場合（Samba書込可）※ 今回は不要
sudo setsebool -P samba_export_all_rw on

# 共有ディレクトリのラベルが必要な場合
sudo semanage fcontext -a -t public_content_rw_t "/srv/nfs(/.*)?"
sudo restorecon -Rv /srv/nfs
```
※ semanage がない場合は sudo dnf install -y policycoreutils-python-utils で導入

### 2. クライアント設定  
2-1. パッケージインストールとマウント設定  
Red Hat 系
```
# パッケージ
sudo dnf install -y nfs-utils

# マウントポイント作成
sudo mkdir -p /mnt/nfs/{public,share}

# マウント（即時）
sudo mount -t nfs 192.168.56.103:/srv/nfs/public /mnt/nfs/public
sudo mount -t nfs 192.168.56.103:/srv/nfs/share /mnt/nfs/share

# 確認
df -hT | grep nfs
```
Debian 系
```
sudo apt install -y nfs-common

sudo mkdir -p /mnt/nfs/{public,share}

sudo mount -t nfs 192.168.56.103:/srv/nfs/public /mnt/nfs/public
sudo mount -t nfs 192.168.56.103:/srv/nfs/share /mnt/nfs/share

df -hT | grep nfs
```
2-2. 永続化設定（任意）  
/etc/fstab に以下を追記： ※ 共有フォルダ自動マウント設定
```
192.168.56.103:/srv/nfs/public  /mnt/nfs/public  nfs  defaults,_netdev  0 0
192.168.56.103:/srv/nfs/share   /mnt/nfs/share   nfs  defaults,_netdev  0 0
```
2-3. マウント確認  
sudo mount -a で一括マウントし df -hT で確認
```
sudo mount -a
df -hT | grep nfs
```
### 3. 特定グループ共有フォルダ用設定  

3-1. サーバ側で使用するグループ・ユーザー作成  
```
sudo groupadd -g 2000 devgroup　※ -g グループID、最後はグループ名
sudo useradd -m -u 2001 -g devgroup stream　※ -m ホームディレクトリ作成、-u ユーザID、-g グループ名指定、最後は作成するユーザ名
sudo useradd -m -u 2002 -g devgroup ubuntu
sudo useradd -m -u 2004 -g devgroup alma
```
3-2. クライアント側も同じ UID/GID で登録ユーザを作成（クライアントサーバごとにそれぞれのユーザ名で作成）
```
sudo groupadd -g 2000 devgroup
sudo useradd -m -u 2001 -g devgroup stream
``````
3-3. クライアント側からアクセス確認
```
su - alma
touch /mnt/nfs/share/test.txt
ls -l /mnt/nfs/share
```

### 4. 備考  
- 特定グループ共有内ファイルを他ユーザーが編集できない場合  
既存ファイルに一括グループ書込権限追加：sudo chmod -R g+w /srv/nfs/share  
新規作成ファイルもグループ書込可にする：sudo chmod g+s /srv/nfs/share　※ NFSはユーザごとにファイル権限が異なるのでディレクトリに setgid ビット設定、g+s でグループID固定で新規ファイルが自動で devgroup 所属になる  
umask を調整して新規ファイルにグループ書込権限付与：クライアント側ユーザが umask=0022 だとグループ書込が外れるのでグループ共同編集時は以下のように変更  
各クライアントユーザ：umask 0006  
恒久化は各ユーザの ~/.bashrc に右記追記：umask 0006  

- Samba ディレクトリと共存させる場合の注意点    
Samba は SMB/CIFS（port 445/139）、NFS は RPC 系ポート群（2049など）を使用するためポート競合は基本的に起きないが、ディレクトリ共存は、ファイルロック競合、属性管理の不整合、SELinux コンテキスト問題などトラブル発生の可能性があるとのこと。Samba が同一ディレクトリを提供する場合、ファイル所有者/パーミッションと SELinux コンテキストが一致していることを要確認（例：/srv/nfs/share を Samba の share と同一にする場合、Samba 用に chown と semanage を適切に設定する）  

- nfs-mountd（rpc.mountd）について  
NFS サーバ上で動作するデーモン、NFS クライアントからのマウント要求を処理してアクセス制御を行う。NFSv4 はプロトコルの仕組み変更のためクライアントとの通信に nfs-mountd は使用していないが、NFSv2 や NFSv3 のクライアントからアクセスを許可している場合、後方互換性のため nfs-mountd も動作必要  

- rpcbind について  
クライアントがネットワーク上の RPC（Remote Procedure Call）サービスに動的に接続するためにサービスのアドレス情報を管理するデーモン。クライアントは rpcbind に問い合わせて RPC サービスが待機しているポート番号を確認して接続を確立する。rpcbind は NFS などで利用されるポート番号111を使用する  
重要性：rpcbind が停止すると RPC サービスへの接続・利用ができなくなる、NFS や NIS のような RPC を利用するサービスには必要不可欠なコンポーネント

### 気になったコマンド
- scp [オプション] コピー元 コピー先：ssh接続ホスト間コピー転送  
-i 鍵ファイル：ssh接続に使用する鍵ファイル指定  
-P ポート番号：接続に使用するポート指定（sshのポートを変更している場合など）  
-p：タイムスタンプやパーミッション保持  
-r：ディレクトリごと再帰的にコピー  
※ 使用例バックアップ送信：scp /etc/hosts root@192.168.56.103:/srv/nfs/system/stream9/

- rsync [オプション] 同期対象 同期先：同期転送（/有：ディレクトリ配下ファイル同期、/無：ディレクトリ同期）、TCP873port  
-a：-rlptgoD と同効果（--recursive --links --perms --times --group --owner --devices）  
-v：処理過程表示  
-r：ディレクトリごと再帰的にコピー  
-e ssh：ssh経由で同期 ※ 使用推奨  
-z：圧縮転送 ※ 転送負荷軽減  
--delete：削除ファイルも同期  
--existing：更新分のみ同期（追加は除く）  
--exclude：除外対象指定  
--version：バージョン確認 ※ バージョン3.4.0未満は深刻な脆弱性が複数あるため即アップデート推奨  
※ 使用例バックアップ同期：rsync -avz -e ssh /etc /srv/nfs/system/stream9_etc/
