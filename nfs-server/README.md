# NFS Server 構成概要（RHEL9.6） 
本構成では Windows と Linux の混在環境を想定し、ファイルサーバとして Samba サーバと NFS サーバの両方を運用しています。  
本リポジトリは **NFS** サーバの構築手順と検証結果を記録しています。

### 特徴比較
| 項目 | Samba | NFS |
|------|--------|-----|
| 対応OS | Windows / macOS / Linux | Linux / UNIX |
| 認証方式 | ユーザ名＋パスワード（SMB認証） | UID / GID（UNIX 標準認証）、IP制御 |
| 権限管理 | ACL（Windows互換） | UNIXパーミッション |
| 通信効率 | やや重い | 軽量・高速 |
| 主な用途 | Windowsクライアント用共有 | Linuxサーバ間連携 |
| 備 考 | Active Directory連携可能 | HAクラスタや仮想環境で安定稼働 |

### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| [NFS_Server_Setup.md](./NFS_Server_Setup.md) | NFS サーバ構築手順 |
| [NFS_Server_Verification.md](./NFS_Server_Verification.md) | 検証結果（接続確認・動作確認） |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| クライアント | stream | Stream 9 | 192.168.56.101 | DNS master / NTPサーバ |
| クライアント | ubuntu | Ubuntu 24.04.3 | 192.168.56.102 | DNS slave / NTPサーバ |
| Samba / **NFS** サーバ | **rhel** | RHEL 9.6 | **192.168.56.103** | Windows・Linux 共有 / **Linux 共有** |
| クライアント | alma | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |

### 共有ディレクトリ構成  
| ディレクトリ | 用途 | アクセス権限 |
|---------------|------|----------------|
| /srv/nfs/share | 共有フォルダ（読み込みのみ）| IP制御（192.168.56.0/24） |
| /srv/nfs/public | 共有フォルダ（読み書き可）| IP制御（192.168.56.0/24）|
