# Infra Lab - Ryo's IT Infrastructure Portfolio 
このリポジトリは サーバ構築・運用の学習記録として作成しました。  
個人検証環境として構築した各種サービスについて、構築手順、設定、検証結果をまとめていきます。



### 構成一覧
  | ディレクトリ | 内 容 |
  |---------------|------|
  | [dns-server](dns-server/) | BINDを使ったDNSサーバ構築と動作確認 |
  | [ntp-server](ntp-server/) | chronyによるNTPサーバ設定 |
  | [samba-server](samba-server/) | Sambaによるファイル共有設定 |
  | [nfs-server](nfs-server/) | NFSによるファイル共有設定 |
  | [apache-server](apache-server/) | httpdによるApacheサーバ設定 |
  | [mysql-server](mysql-server/) | mysqldによるDBサーバ設定 |
  | [php-server](php-server/) | phpサーバ設定 |
  | [zabbix](zabbix/) | Zabbixによる監視環境構築 |

### 学習環境
  | 項 目 | 内 容 |
  |------|------|
  | ホストOS | Windows 11 |
  | WSL | WSL2 Ubuntu 24.04 |
  | 仮想化環境 | VirtualBox 7.1.10 |
  | ゲストOS | CentOSStream9 / Ubuntu24.04.3 / RHEL9.6 / AlmaLinux9.6 / KaliLinux2025.3 / Debian12 |
  | IPアドレス |192.168.56.101 ～ 105 / 192.168.56.110 |
  | ネットワークアダプタ | NAT + Host-Only |
  | エディタ | サクラエディタ Ver.2.4.2.6048 |
  | バージョン管理 | Git for Windows v2.51.0 / GitHub |



### 各 OS の役割（カテゴリ別）


管理・オーケストレーション
| O S | 役 割 |
|----|------|
| WSL2 Ubuntu 24.04 | CA 認証局（SSH CA）、Ansible 管理ホスト |


基盤サービス（DNS / NTP）
| O S | 役 割 |
|----|------|
| CentOS Stream 9 | DNS（master）サーバ、NTP サーバ |
| Ubuntu 24.04.3 | DNS（slave）サーバ、NTP サーバ |


ストレージ・ファイル共有
| O S | 役 割 |
|----|------|
| RHEL 9.6 | Samba サーバ、NFS サーバ |


Web / DB / 監視（LAMP / Zabbix）
| O S | 役 割 |
|----|------|
| AlmaLinux 9.6 | LAMP 環境（Apache / MySQL / PHP）および Zabbix サーバ |


セキュリティ検証
| O S | 役 割 |
|----|------|
| Kali Linux 2025.3 | セキュリティ検証・ペネトレーションテスト |


セキュアなリモートアクセス・システム管理サーバ
| O S | 役 割 |
|----|------|
| Debian 12 | ZeroTier エンドポイント、Fail2ban、msmtp（メール通知）、rsyslog（ログ集約）|


モバイルクライアント
| O S | 役 割 |
|----|------|
| Android（Termux）| ZeroTier クライアント、SSH ターミナル |

### 今後の目標
- 自宅サーバの構築情報を順次アップロード  
- Zabbixへの監視対象サービスの追加
- 監視・ログ収集・バックアップの連携実験
- Kali Linux によるセキュリティ検証
- サーバ構築の一部自動化（Shellスクリプト化）

<img width="1493" height="849" alt="image" src="https://github.com/user-attachments/assets/bff36017-aae3-4021-9f15-c5bb7cc082c1" />
<img width="1919" height="617" alt="image" src="https://github.com/user-attachments/assets/e442a202-4192-42a8-8044-0e4ca95370a1" />
<img width="1919" height="726" alt="image" src="https://github.com/user-attachments/assets/926c98b2-5b32-4332-a60f-3b4539b73cf8" />
<img width="1919" height="678" alt="image" src="https://github.com/user-attachments/assets/3e21f12f-cd9f-46b2-bc99-604441ec6b94" />

