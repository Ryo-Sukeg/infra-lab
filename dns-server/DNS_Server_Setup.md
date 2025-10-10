# DNS Server 構築手順記録

## 構成概要
| 項目 | 内容 |
|------|------|
| ドメイン | lab.lan |
| マスターDNS | CentOS Stream 9.6（192.168.56.101） |
| スレーブDNS | Ubuntu 24.04（192.168.56.102） |
| クライアント | RHEL 9.6 / AlmaLinux 9.6 |
| サービス | BIND9 |
| 役割 | 内部向け正引き・逆引きDNSサーバ |
---

## 手順
### 1. マスターDNSサーバ構築 (CentOS Stream 9.6)  
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
listen-on port 53 { 127.0.0.1; 192.168.56.101; };
allow-query     { localhost; 192.168.56.0/24; };

recursion yes;

zone "lab.lan" IN {
    type master;
    file "/var/named/lab.lan.zone";
    allow-update { none; };
};

zone "56.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/56.168.192.rev";
    allow-update { none; };
};

};
```
1-3. ゾーンファイル作成
```
sudo vi /var/named/lab.lan.zone
```
内容：
```
$TTL 86400
@   IN  SOA master.lab.lan. root.lab.lan. (
        2025100701  ; Serial
        3600        ; Refresh
        900         ; Retry
        604800      ; Expire
        86400 )     ; Minimum
    IN  NS  master.lab.lan.
    IN  NS  slave.lab.lan.
master  IN  A   192.168.56.101
slave   IN  A   192.168.56.102
client1 IN  A   192.168.56.103
client2 IN  A   192.168.56.104
```
逆引きゾーン：
```
sudo vi /var/named/56.168.192.rev
```
内容：
```
$TTL 86400
@   IN  SOA master.lab.lan. root.lab.lan. (
        2025100701
        3600
        900
        604800
        86400 )
    IN  NS  master.lab.lan.
    IN  NS  slave.lab.lan.
101 IN PTR master.lab.lan.
102 IN PTR slave.lab.lan.
103 IN PTR client1.lab.lan.
104 IN PTR client2.lab.lan.
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
sudo named-checkzone 56.168.192.in-addr.arpa /var/named/56.168.192.rev
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
    file "/var/lib/bind/lab.lan.zone";
};

zone "56.168.192.in-addr.arpa" {
    type slave;
    masters { 192.168.56.101; };
    file "/var/lib/bind/56.168.192.rev";
};
```
2-3. 起動・確認
```
sudo systemctl enable --now bind9
sudo systemctl status bind9
ls -l /var/lib/bind/
```
→ ゾーンファイルが自動転送されていれば成功
```
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
