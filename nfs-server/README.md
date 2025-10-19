# NFS Server 構成概要（RHEL9.6） 
本リポジトリは `NFS` サーバの構築手順と検証結果を記録しています。同一ホスト上で Samba・NFS 両対応の共有ファイルサーバ環境を構築・検証することを目的としています。

#### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| [NFS_Server_Setup.md](./NFS_Server_Setup.md) | NFS サーバ構築手順 |
| [NFS_Server_Verification.md](./NFS_Server_Verification.md) | 検証結果（接続確認・動作確認） |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| Linux クライアント | stream | Stream 9 | 192.168.56.101 | DNS master / NTPサーバ |
| Linux クライアント | ubuntu | Ubuntu 24.04.3 | 192.168.56.102 | DNS slave / NTPサーバ |
| Samba / *NFS* サーバ | *rhel* | RHEL 9.6 | *192.168.56.103* | Samba共有と同一ディレクトリ利用 |
| Linux クライアント | alma | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |
| Windows クライアント | win11-test | Windows 11 | 172.21.100.x | 共有アクセス確認用 |

### 共有ディレクトリ構成  
| ディレクトリ | 用途 | アクセス権限 |
|---------------|------|----------------|
| /srv/nfs/share | 認証共有フォルダ | 認証ユーザ専用 |
| /srv/nfs/public | 共有フォルダ | 全クライアントアクセス可|
