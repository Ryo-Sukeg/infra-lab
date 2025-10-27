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
# 特定グループ共有（特定 UID/GID のみ書込可）

sudo mkdir -p /srv/nfs/share
sudo chmod -R 770 /srv/nfs/share
sudo useradd nfsuser	# 共有用ユーザ作成
sudo passwd nfsuser
sudo chown -R nfsuser:nfsuser /srv/nfs/share
```
```
# 管理者専用（管理者のみ書込可）

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
ro：読み取り専用（公開用フォルダ向け）  
sync：書き込み時に同期、データを確実にディスクへ書き込んでから応答  
root_squash：クライアント root を匿名ユーザ nobody に置き換え（権限昇格防止）  
no_subtree_check：サブツリーチェック無効化で転送高速化（必要に応じて追加）  

1-4. 設定反映と確認  
エクスポート反映・確認（一覧）
```
sudo exportfs -rv
```
マウント状態確認
```
sudo showmount -e
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
sudo firewall-cmd --list-all --zone=public
```
1-7. SELinux 設定  
有効でも読み書きできるようブール値を設定
```
# NFS書き込み許可（NFS共有に対する一般的設定）
sudo setsebool -P nfs_export_all_rw on

# Samba と同一ディレクトリを使う場合（Samba書き込み許可）※ 今回は不要
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
sudo mount -t 192.168.56.103:/srv/nfs/public /mnt/nfs/public
sudo mount -t 192.168.56.103:/srv/nfs/share /mnt/nfs/share

# 確認
df -hT | grep nfs
```
Debian 系
```
sudo apt update
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
192.168.56.103:/srv/nfs/system   /mnt/nfs/system   nfs  defaults,_netdev  0 0
```
2-3. マウント確認  
sudo mount -a で一括マウントし df -hT で確認
```
sudo mount -a
df -hT | grep nfs
```

### 3. 備考
- Samba ディレクトリと共存させる場合の注意点    
Samba が同一ディレクトリを提供する場合、ファイル所有者 / パーミッションと SELinux コンテキストが一致していることを確認する。
例：/srv/nfs/share を Samba の share と同一にする場合、Samba 用に chown と semanage を適切に設定する。  
また、Samba は SMB / CIFS（ポート445/139）、NFS は RPC 系のポート群（2049 など）を使用するためポート競合は基本的に起きない。

- nfs-mountd（rpc.mountd）について  
NFS サーバ上で動作するデーモン、NFS クライアントからのマウント要求を処理してアクセス制御を行う。NFSv4 はプロトコルの仕組み変更のためクライアントとの通信に nfs-mountd は使用していない。しかし、NFSv2 や NFSv3 のクライアントからアクセスを許可している場合、後方互換性のため nfs-mountd も動作が必要。  
主な機能  
マウント要求処理：クライアントから共有リソース（ディレクトリ）へのマウント要求を受け付け。  
アクセス許可確認：マウント要求が来るとサーバーの /etc/exports ファイルに記述されたリストを参照し、要求元クライアントがそのファイルシステムをマウントする権限を持っているかチェックする。  
ファイルハンドルの発行：許可されたクライアントには要求されたディレクトリを識別するためのファイルハンドルを返す。このファイルハンドルはNFSクライアントがサーバー上のファイルにアクセスするためのポインターのような役割を果たす。  
マウント情報記録：クライアントからのマウント要求を受けると /var/lib/nfs/rmtab ファイルにマウント情報を記録する。  

- rpcbind について  
クライアントがネットワーク上の RPC（Remote Procedure Call）サービスに動的に接続するためにサービスのアドレス情報を管理するデーモン。クライアントは rpcbind に問い合わせて RPC サービスが待機しているポート番号を確認して接続を確立する。rpcbind は NFS などで利用されるポート番号111を使用する。  
役割  
リモートで実行されるサービス（サーバ）がリクエストを受けるために使用するポート番号をクライアントに伝える。  
プログラム番号とバージョン番号のマッピング：RPC プログラムの番号とバージョン番号をサーバー上の汎用アドレス（トランスポートに依存したアドレス）にマップする。   ﻿  
動的接続の実現：クライアントが特定サービスのアドレスを知らなくても rpcbind を介してサービスに接続できるようになる。       
ブロードキャスト RPC のサポート：クライアントからのブロードキャストメッセージを rpcbind がローカルの RPC サービスに中継する。   ﻿  
サービス登録：他の RPC サービスは rpcbind に自身のアドレスを登録して利用可能にする。  
重要性  
rpcbind が停止すると RPC サービスへの接続・利用ができなくなる。  ﻿  
NFS や NIS のような RPC を利用するサービスには必要不可欠なコンポーネント。  ﻿
