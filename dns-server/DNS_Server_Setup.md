# DNS Server 構築手順記録

## 構成概要
| 項目 | 内容 |
|------|------|
| DNS | Master : CentOS Stream 9（192.168.56.101）/ Slave : Ubuntu 24.04（192.168.56.102）|
| ドメイン | lab.lan |
| サービス | BIND9 |
| 役割 | 内部向け正引き・逆引きDNSサーバ |
| クライアント | RHEL 9.6 / AlmaLinux 9.6 |
---

## 手順
### 1. マスターDNSサーバ構築 (CentOS Stream 9)  
1-1. BINDインストール
```
sudo dnf install -y bind bind-utils
```
1-2. 設定ファイル編集 /etc/named.conf
```
sudo vi /etc/named.conf
```
以下のように編集（変更・追記部分のみ抜粋）：
```
options {
    listen-on port 53 { 127.0.0.1; 192.168.56.101; };
    allow-query     { localhost; 192.168.56.0/24; };
    recursion yes;

    forwarders { 8.8.8.8; 1.1.1.1; };
    forward only;

    directory "/var/named";
};

zone "lab.lan" IN {
    type master;
    file "lab.lan.zone";
    allow-transfer { 192.168.56.102; };
};

zone "56.168.192.in-addr.arpa" IN {
    type master;
    file "56.168.192.zone";
    allow-transfer { 192.168.56.102; };
};
```
1-3. ゾーンファイル作成
```
sudo vi /var/named/lab.lan.zone
```
内容：
```
$TTL 86400
@   IN  SOA     dns.lab.lan. root.lab.lan. (
        2025100601 ; Serial（変更ごとに+1）
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Minimum TTL

@       IN  NS      dns.lab.lan.
dns     IN  A       192.168.56.101
stream  IN  A       192.168.56.101
ubuntu  IN  A       192.168.56.102
rhel    IN  A       192.168.56.103
alma    IN  A       192.168.56.104
```
逆引きゾーン：
```
sudo vi /var/named/56.168.192.zone
```
内容：
```
$TTL 86400
$TTL 86400
@   IN  SOA     dns.lab.lan. root.lab.lan. (
        2025100501
        3600
        1800
        604800
        86400 )

@       IN  NS      dns.lab.lan.
101     IN  PTR     stream.lab.lan.
102     IN  PTR     ubuntu.lab.lan.
103     IN  PTR     rhel.lab.lan.
104     IN  PTR     alma.lab.lan.
```
1-4. firewalld・SELinux設定
```
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
sudo setsebool -P named_write_master_zones on
```
1-5. サービス起動と動作確認
```
sudo systemctl enable --now named
sudo systemctl status named
sudo named-checkconf
sudo named-checkzone lab.lan /var/named/lab.lan.zone
sudo named-checkzone 56.168.192.in-addr.arpa /var/named/56.168.192.zone
```
### 2. スレーブDNSサーバ構築 (Ubuntu 24.04)
2-1. BINDインストール
```
sudo apt install -y bind9 bind9-utils
```
2-2. 設定ファイル編集 /etc/bind/named.conf.local
```
zone "lab.lan" {
    type slave;
    masters { 192.168.56.101; };
    file "/var/cache/bind/lab.lan.zone";
};

zone "56.168.192.in-addr.arpa" {
    type slave;
    masters { 192.168.56.101; };
    file "/var/cache/bind/56.168.192.in-addr.arpa.zone";
};
```
2-3. 起動・確認
```
sudo systemctl enable --now bind9
sudo systemctl status bind9
ls -l /var/lib/bind/
```
→ ゾーンファイルが自動転送されていれば成功
### 3. クライアント設定 (RHEL/Alma)
3-1. /etc/resolv.conf を編集
```
sudo vi /etc/resolv.conf
```
内容：
```
nameserver 192.168.56.101
nameserver 192.168.56.102
search lab.lan
```
3-2. 動作確認
```
nslookup master.lab.lan
dig slave.lab.lan
dig -x 192.168.56.101
```
### 4. 備考
- 両DNSが正引き・逆引きともに応答することを確認
- Serial 番号を上げるとスレーブへ自動転送される
- BINDのゾーン定義は /etc/named.conf or /etc/bind/named.conf.local に分けると分かりやすい
- ファイアウォールはUDP/TCPの53番ポートを許可
- スレーブが同期しない場合は /var/log/messages or /var/log/syslog を確認
