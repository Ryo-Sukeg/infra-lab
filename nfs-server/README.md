# NFS Server 構成概要（RHEL9.6） 
本構成では Windows と Linux の混在環境を想定し、ファイルサーバとして Samba サーバと NFS サーバの両方を運用しています。  
本リポジトリは **NFS** サーバの構築手順と検証結果を記録しています。  

### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| [NFS_Server_Setup.md](./NFS_Server_Setup.md) | NFS サーバ構築手順 |
| [NFS_Server_Verification.md](./NFS_Server_Verification.md) | 検証結果（接続確認・動作確認）|

### 特徴比較
| 項 目 | Samba | NFS |
|------|--------|-----|
| 対応OS | Windows / macOS / Linux | Linux / UNIX |
| 認証方式 | ユーザ名＋パスワード（SMB 認証） | UID / GID（UNIX 標準認証）|
| 権限管理 | ACL（Windows 互換） | UNIX パーミッション |
| 通信効率 | やや重い | 軽量・高速 |
| 主な用途 | Windows クライアント用共有 | Linux サーバ間連携 |
| 備 考 | Active Directory 連携可能 | HAクラスタや仮想環境で安定稼働、firewall・IP 制御、Kerberos 認証追加可能 |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| クライアント | stream | Stream 9 | 192.168.56.101 | DNS master / NTPサーバ |
| クライアント | ubuntu | Ubuntu 24.04.3 | 192.168.56.102 | DNS slave / NTPサーバ |
| Samba / **NFS** サーバ | **rhel** | RHEL 9.6 | **192.168.56.103** | Windows・Linux 共有 / **Linux 共有** |
| クライアント | alma | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |

### 共有ディレクトリ構成  
| ディレクトリ | 用 途 | アクセス権限 | 備 考 |
|---------------|------|----------------|-------|
| /srv/nfs/public | 公開共有（閲覧専用）| IP 制御（192.168.56.0/24）| Linux 共有 |
| /srv/nfs/share | 特定グループ共有（書込可）| 特定 UID/GID、IP 制御（192.168.56.0/24）| Linux 共有 |
| /srv/nfs/system | バックアップなど（管理者専用） | root、IP 制御（192.168.56.103）| 管理サーバ専用 |

※ samba 共有ディレクトリと別ディレクトリにした理由：ファイルロック競合・属性管理の不整合・SELinux コンテキスト問題などのトラブル回避
