# Samba Server 構成概要（RHEL9.6） 
本構成では Windows と Linux の混在環境を想定し、ファイルサーバとして Samba サーバと NFS サーバの両方を運用しています。  
本リポジトリは **Samba** サーバの構築手順と検証結果を記録しています。
Windows / Linux 両クライアントから同一共有ディレクトリへアクセス可能な構成にしています。

### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| [Samba_Server_Setup.md](./Samba_Server_Setup.md) | Samba サーバ構築手順 |
| [Samba_Server_Verification.md](./Samba_Server_Verification.md) | 検証結果（接続確認・動作確認） |

### 特徴比較
| 項 目 | Samba | NFS |
|------|--------|-----|
| 対応OS | Windows / macOS / Linux | Linux / UNIX |
| 認証方式 | ユーザ名＋パスワード（SMB認証） | UID / GID（UNIX 標準認証）、IP制御 |
| 権限管理 | ACL（Windows互換） | UNIXパーミッション |
| 通信効率 | やや重い | 軽量・高速 |
| 主な用途 | Windowsクライアント用共有 | Linuxサーバ間連携 |
| 備 考 | Active Directory連携可能 | HAクラスタや仮想環境で安定稼働、firewall・IP 制御、Kerberos 認証追加可能 |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| Linux クライアント | stream | Stream 9 | 192.168.56.101 | DNS master / NTPサーバ |
| Linux クライアント | ubuntu | Ubuntu 24.04.3 | 192.168.56.102 | DNS slave / NTPサーバ |
| **Samba** / NFS サーバ | **rhel** | RHEL 9.6 | **192.168.56.103** | **Windows・Linux 共有** / Linux 共有 |
| Linux クライアント | alma | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |
| Windows クライアント | win11 | Windows 11 | 172.21.100.x | 共有アクセス確認用 |

### 共有ディレクトリ構成  
| ディレクトリ | 用 途 | アクセス権限 | 備 考 |
|---------------|------|----------------|------|
| /mnt/samba/share | メイン共有フォルダ | 読み書き可（認証ユーザのみ）| Linux・Windows共有 |
| /mnt/samba/public | 共有フォルダ | 読み書き可（全ユーザ） | Linux・Windows共有 |


<img width="1752" height="843" alt="image" src="https://github.com/user-attachments/assets/b07a973e-fa89-46cb-a45b-d18afe610f92" />



<img width="1753" height="840" alt="image" src="https://github.com/user-attachments/assets/20b867a5-4999-4aeb-a3a6-48eef8e2deee" />
