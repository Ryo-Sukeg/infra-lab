# DNS Server 構築・検証記録  
本リポジトリでは `BIND9` を使用した DNS サーバ構築および検証の手順を記録しています。

### 構成情報  
| 役割 | ホスト名 | O S | IPアドレス | 主なサービス |  
|------|-----------|----|-------------|---------------|  
| Master DNS | stream9.6 | CentOS Stream 9 | 192.168.56.101 | BIND (named) |  
| Slave DNS | ubuntu24 | Ubuntu 24.04 | 192.168.56.102 | BIND (named) |  
| Client 1 | rhel9.6 | RHEL 9.6 | 192.168.56.103 | dig / nslookup |  
| Client 2 | alma9.6 | AlmaLinux 9.6 | 192.168.56.104 | dig / nslookup |  

### ファイル構成  
| ファイル名	| 内  容 |  
|-----------|----|  
| build_guide.md	| BINDの構築手順 ( master/slave設定 ) |  
| verification.md | digコマンドを使った検証結果記録 |  

### 目的    
ローカルネットワーク内にBINDを利用したDNSサーバを構築し、内部ドメイン名解決を行える環境を作成する。
- 内部ドメイン例 : lab.local  
- 主な検証内容 : 正引き／逆引きの動作確認、セカンダリDNSへのゾーン転送、 フェイルオーバー時の名前解決確認  

### 備考  
- SELinux は permissive または disabled に設定して動作検証を行う  
- Firewall は dns (TCP/UDP 53) を許可する  
- 検証環境は VirtualBox + NATネットワークで構築  
