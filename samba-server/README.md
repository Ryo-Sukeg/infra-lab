# Samba Server 構成概要（RHEL9.6） 
本リポジトリは `Samba` サーバの構築手順と検証結果を記録しています。
Windows / Linux 両クライアントから同一共有ディレクトリへアクセス可能な構成にしています。

### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| [Samba_Server_Setup.md](./Samba_Server_Setup.md) | Samba サーバ構築手順 |
| [Samba_Server_Verification.md](./Samba_Server_Verification.md) | 検証結果（接続確認・動作確認） |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| Linux クライアント | stream9.6 | Stream 9 | 192.168.56.101 | DNS master / NTPサーバ |
| Linux クライアント | ubuntu24 | Ubuntu 24.04.3 | 192.168.56.102 | DNS slave / NTPサーバ |
| Samba / NFS サーバ | rhel9.6 | RHEL 9.6 | 192.168.56.103 | NFS共有と同一ディレクトリ利用 |
| Linux クライアント | alma9.6 | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |
| Windows クライアント | win11-test | Windows 11 | 172.21.100.x | 共有アクセス確認用 |

### 共有ディレクトリ構成  
| ディレクトリ | 用途 | アクセス権限 | 備考 |
|---------------|------|----------------|------|
| /srv/samba/share | メイン共有フォルダ | 読み書き可（認証ユーザのみ）| Linux / Windows共通 |
| /srv/public | 共有フォルダ | 読み書き可（全ユーザ） | Linux / Windows共通 |

<img width="1572" height="882" alt="image" src="https://github.com/user-attachments/assets/24be3ec4-a61b-412b-9a58-f3d883a897ab" />
