# Samba Server 構成概要 ( RHEL9.6 )  
本リポジトリは RHEL9.6 上で構築した Samba サーバの設定情報と検証結果をまとめたものです。
Windows / Linux 両クライアントから同一共有ディレクトリへアクセス可能な構成にしています。

### ファイル構成  
| ファイル名 | 内 容 |
|-------------|------|
| Samba_Server_Setup.md | Samba サーバ構築手順 |
| Samba_Server_Verification.md | 検証結果（接続確認・動作確認ログ） |

### 環境構成  
| 役 割 | ホスト | O S | IPアドレス | 備 考 |
|------|---------|----|-------------|------|
| NTP・DNS サーバ | stream9.6 | Stream9 | 192.168.56.101 | 時刻同期・DNS master |
| NTP・DNS サーバ | ubuntu24 | Ubuntu 24.04 | 192.168.56.102 | 時刻同期・DNS slave |
| Samba・NFS サーバ | rhel9.6 | RHEL9.6 | 192.168.56.103 | NFS共有と同一ディレクトリ利用 |
| LAMP・Zabbix サーバ | alma9.6 | AlmaLinux9.6 | 192.168.56.104 | Webサーバ環境 |
| Windows クライアント | win11-test | Windows 11 | 192.168.56.x | 共有アクセス確認用 |

### 共有ディレクトリ構成  
/srv/samba/share
