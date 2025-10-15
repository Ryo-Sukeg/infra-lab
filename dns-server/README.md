# DNS Server 構築・検証記録  
本リポジトリは `BIND9` を使用した DNS サーバの構築手順と検証結果を記録しています。DNSサーバは `Master Server` と `Slave Server` を構築し冗長化の確認も行っています。

### ファイル構成  
| ファイル名	| 内 容 |  
|-----------|----|  
| [DNS_Server_Setup.md](./DNS_Server_Setup.md) | BINDの構築手順（master/slave設定）|  
| [DNS_Server_verification.md](./DNS_Server_Verification.md) | digコマンドを使った検証結果記録 |  

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 主なサービス |  
|------|-----------|----|-------------|---------------|  
| **DNS master** | **stream** | CentOS Stream 9 | **192.168.56.101** | **BIND（named）/ chrony** |  
| **DNS slave** | **ubuntu** | Ubuntu 24.04.3 | **192.168.56.102** | **BIND（named）/ chrony** |  
| クライアント | rhel | RHEL 9.6 | 192.168.56.103 | samba / NFS |  
| クライアント | alma | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix |  

### 目的    
ローカルネットワーク内にBINDを利用したDNSサーバを構築し、内部ドメイン名解決を行える環境を作成する。
- 内部ドメイン : lab.lan  
- 主な検証内容 : 正引き／逆引きの動作確認、セカンダリDNSへのゾーン転送、 フェイルオーバー時の名前解決確認  

### 備考  
- SELinux は permissive または disabled に設定して動作検証を行う  
- Firewall は dns（TCP/UDP 53）を許可する  
- 検証環境は VirtualBox + NATネットワークで構築  

<img width="1662" height="761" alt="image" src="https://github.com/user-attachments/assets/2d8fd891-3181-4276-82d5-0df6ccb9cf6f" />
