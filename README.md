# Infra Lab - Ryo's IT Infrastructure Portfolio 
このリポジトリは サーバ構築・運用の学習記録として作成しました。  
個人検証環境で構築した各種サービス（DNS / NTP / Samba / LAMP環境 / Zabbixなど）の構築手順、設定、検証結果をまとめていきます。

## 目的
- Linuxサーバ構築・運用の実践スキルを身につける  
- 手順・設定・検証結果を体系的にまとめる  
- 構築環境の記録・復習として整理  

## 構成一覧
  | ディレクトリ | 内容 |
  |---------------|------|
  | [dns-server](dns-server/) | BINDを使ったDNSサーバ構築と動作確認 |
  | [ntp-server](ntp-server/) | chronyによるNTPサーバ設定 |
  | [samba-server](samba-server/) | Sambaによるファイル共有設定 |
  | [nfs-server](sfs-server/) | Nfsによるファイル共有設定 |
  | [apache-server](apache-server/) | httpdによるApacheサーバ設定 |
  | [mysql-server](mysql-server/) | mysqldによるDBサーバ設定 |
  | [php-server](php-server/) | phpサーバ設定 |
  | [zabbix](zabbix/) | Zabbixによる監視環境構築 |
  | [notes](notes/) | Linux基本コマンド・トラブル対応メモ |

## 学習環境
  | 項目 | 内容 |
  |------|------|
  | ホストOS | Windows 11 |
  | 仮想化環境 | VirtualBox 7.1.10 |
  | ゲストOS | CentOS_Stream 9 / Ubuntu 24.04 / RHEL 9.6 / AlmaLinux 9.6 |
  | IPアドレス | 192.168.56.101 / 192.168.56.102 / 192.168.56.103 /  192.168.56.104 |
  | ネットワークアダプタ | NAT + Host-Only |
  | エディタ | SakuraEditor |
  | バージョン管理 | Git for Windows v2.51.0 / GitHub |

## 今後の目標
- 自宅サーバの構築情報を順次アップロード  
- Zabbixへの監視対象サービスの追加
- 監視・ログ収集・バックアップの連携実験  
- サーバ構築の一部自動化（Shellスクリプト化）

<img width="1493" height="849" alt="image" src="https://github.com/user-attachments/assets/bff36017-aae3-4021-9f15-c5bb7cc082c1" />
<img width="1919" height="617" alt="image" src="https://github.com/user-attachments/assets/e442a202-4192-42a8-8044-0e4ca95370a1" />
<img width="1919" height="726" alt="image" src="https://github.com/user-attachments/assets/926c98b2-5b32-4332-a60f-3b4539b73cf8" />
<img width="1919" height="678" alt="image" src="https://github.com/user-attachments/assets/3e21f12f-cd9f-46b2-bc99-604441ec6b94" />

