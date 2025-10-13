# DNS Server 検証結果記録

## 構成概要

| サーバ種別 | O S | ホスト名 | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| DNS | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | マスターサーバ |
| DNS | Ubuntu 24.04.3 | ubuntu.lab.lan | 192.168.56.102 | スレーブサーバ |
| client | RHEL 9.6 | rhel.lab.lan | 192.168.56.103 | 正引き・逆引き確認 |
| client | AlmaLinux 9.6 | alma.lab.lan| 192.168.56.104 | フェイルオーバーテスト |

---

### 1. マスター・スレーブの状態確認

stream.lab.lan（master DNS）

```bash
sudo systemctl status named
sudo rndc status
```
出力例：
```
sudo systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2025-10-10 16:54:56 JST; 1min 48s ago
    Process: 783 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-ch>
    Process: 841 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 855 (named)
      Tasks: 10 (limit: 10651)
     Memory: 31.3M (peak: 31.7M)
        CPU: 430ms
     CGroup: /system.slice/named.service
             mq855 /usr/sbin/named -u named -c /etc/named.conf

10月 10 16:54:56 Stream9.6 named[855]: zone 0.in-addr.arpa/IN: loaded serial 0
10月 10 16:54:56 Stream9.6 named[855]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
10月 10 16:54:56 Stream9.6 named[855]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip>
10月 10 16:54:56 Stream9.6 named[855]: zone localhost.localdomain/IN: loaded serial 0
10月 10 16:54:56 Stream9.6 named[855]: all zones loaded
10月 10 16:54:56 Stream9.6 named[855]: running
10月 10 16:54:56 Stream9.6 systemd[1]: Started Berkeley Internet Name Domain (DNS).
10月 10 16:54:56 Stream9.6 named[855]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance time>
10月 10 16:54:56 Stream9.6 named[855]: managed-keys-zone: Key 38696 for zone . is now trusted (acceptance time>
10月 10 16:54:57 Stream9.6 named[855]: listening on IPv6 interface enp0s3, fd17:625c:f037:2:a00:27ff:fe1c:1875>[
------------------------------------------------------------------------------------------------------------------
sudo rndc status
version: BIND 9.16.23-RH (Extended Support Version) <id:fde3b1f>
running on Stream9.6: Linux x86_64 5.14.0-620.el9.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Sep 26 01:13:23 UTC 2025
boot time: Fri, 10 Oct 2025 07:54:55 GMT
last configured: Fri, 10 Oct 2025 07:54:56 GMT
configuration file: /etc/named.conf
CPUs found: 2
worker threads: 2
UDP listeners per interface: 2
number of zones: 8 (0 automatic)
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is OFF
recursive clients: 0/900/1000
tcp clients: 0/150
TCP high-water: 0
server is up and running
```
ubuntu.lab.lan（slave DNS）
```
sudo systemctl status named
sudo tail -n 5 /var/log/syslog
```
出力例：
```
sudo systemctl status named
● named.service - BIND Domain Name Server
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-10-10 17:04:25 JST; 2min 40s ago
       Docs: man:named(8)
   Main PID: 1283 (named)
     Status: "running"
      Tasks: 8 (limit: 2216)
     Memory: 28.5M (peak: 28.9M)
        CPU: 536ms
     CGroup: /system.slice/named.service
             mq1283 /usr/sbin/named -f -u bind

10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:500:2d::d#53
10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:500:9f::42#53
10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:7fd::1#53
10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:500:2::c#53
10月 10 17:04:25 Ubuntu24 named[1283]: network unreachable resolving './NS/IN': 2001:500:a8::e#53
10月 10 17:04:25 Ubuntu24 named[1283]: managed-keys-zone: Key 20326 for zone . is now trusted (acceptance >
10月 10 17:04:25 Ubuntu24 named[1283]: managed-keys-zone: Key 38696 for zone . is now trusted (acceptance >
10月 10 17:04:26 Ubuntu24 named[1283]: listening on IPv6 interface enp0s3, fd17:625c:f037:2:f7ae:2c82:5f0c>
10月 10 17:04:26 Ubuntu24 named[1283]: listening on IPv6 interface enp0s3, fd17:625c:f037:2:a00:27ff:fe05:>
------------------------------------------------------------------------------------------------------------------
sudo tail -n 5 /var/log/syslog
2025-10-10T17:09:29.084710+09:00 Ubuntu24 systemd[1]: Started systemd-timedated.service - Time & Date Service.
2025-10-10T17:09:59.121244+09:00 Ubuntu24 systemd[1]: systemd-timedated.service: Deactivated successfully.
2025-10-10T17:10:01.469091+09:00 Ubuntu24 systemd[1]: Starting sysstat-collect.service - system activity accounting tool...
2025-10-10T17:10:01.562118+09:00 Ubuntu24 systemd[1]: sysstat-collect.service: Deactivated successfully.
2025-10-10T17:10:01.564048+09:00 Ubuntu24 systemd[1]: Finished sysstat-collect.service - system activity accounting tool.
```
### 2. 正引き・逆引き確認
クライアント（RHEL9.6）
```
dig stream.lab.lan
dig ubuntu.lab.lan
dig -x 192.168.56.104
```
出力例：
```
; <<>> DiG 9.16.23-RH <<>> stream.lab.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28104
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9362bd0ec30bd3960100000068e9d5eb4124a68a67e01934 (good)
;; QUESTION SECTION:
;stream.lab.lan.                        IN      A

;; ANSWER SECTION:
stream.lab.lan.         86400   IN      A       192.168.56.101

;; Query time: 1 msec
;; SERVER: 192.168.56.101#53(192.168.56.101)
;; WHEN: Sat Oct 11 12:58:35 JST 2025
;; MSG SIZE  rcvd: 87
-------------------------------------------------------------------------
; <<>> DiG 9.16.23-RH <<>> ubuntu.lab.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58040
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 9424d7a4493e68c30100000068e9d644d81159bcdb7db1e4 (good)
;; QUESTION SECTION:
;ubuntu.lab.lan.                        IN      A

;; ANSWER SECTION:
ubuntu.lab.lan.         86400   IN      A       192.168.56.102

;; Query time: 2 msec
;; SERVER: 192.168.56.101#53(192.168.56.101)
;; WHEN: Sat Oct 11 13:00:04 JST 2025
;; MSG SIZE  rcvd: 87
-------------------------------------------------------------------------
; <<>> DiG 9.16.23-RH <<>> -x 192.168.56.104
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8431
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0b9357c0cbd6840d0100000068e9d6abc283e9df2f40e8d4 (good)
;; QUESTION SECTION:
;104.56.168.192.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
104.56.168.192.in-addr.arpa. 86400 IN   PTR     alma.lab.lan.

;; Query time: 2 msec
;; SERVER: 192.168.56.101#53(192.168.56.101)
;; WHEN: Sat Oct 11 13:01:47 JST 2025
;; MSG SIZE  rcvd: 110
```
### 3. フェイルオーバーテスト
マスター停止時（スレーブ応答確認）
```
# master側で停止
sudo systemctl stop named

# クライアントから確認
dig rhel.lab.lan
```
出力例：
```
; <<>> DiG 9.16.23-RH <<>> rhel.lab.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29610
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a1ca7d4632aa33920100000068e9df468d1e31f00c7c51c8 (good)
;; QUESTION SECTION:
;rhel.lab.lan.                  IN      A

;; ANSWER SECTION:
rhel.lab.lan.           86400   IN      A       192.168.56.103

;; Query time: 3 msec
;; SERVER: 192.168.56.102#53(192.168.56.102)    // 問い合わせ先DNS
;; WHEN: Sat Oct 11 13:38:30 JST 2025
;; MSG SIZE  rcvd: 85
```
スレーブ停止時（マスター応答確認）
```
# slave側で停止
sudo systemctl stop named

# クライアントから確認
dig alma.lab.lan
```
出力例：
```
; <<>> DiG 9.16.23-RH <<>> alma.lab.lan
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46387
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: f2f4a899395b48a00100000068e9dfa7279687645e40bc29 (good)
;; QUESTION SECTION:
;alma.lab.lan.                  IN      A

;; ANSWER SECTION:
alma.lab.lan.           86400   IN      A       192.168.56.104

;; Query time: 4 msec
;; SERVER: 192.168.56.101#53(192.168.56.101)    // 問い合せ先DNS
;; WHEN: Sat Oct 11 13:40:07 JST 2025
;; MSG SIZE  rcvd: 85
```
外部ドメイン検索
```
dig amazon

; <<>> DiG 9.16.23-RH <<>> amazon
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42904
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 224fa223a2e79c650100000068e9e0f490f1b1c9960ce947 (good)
;; QUESTION SECTION:
;amazon.                                IN      A

;; AUTHORITY SECTION:
amazon.                 60      IN      SOA     dns1.nic.amazon. hostmaster.nominet.org.uk. 1760136705 900 300 2419200 60

;; Query time: 54 msec
;; SERVER: 192.168.56.102#53(192.168.56.102)
;; WHEN: Sat Oct 11 13:45:40 JST 2025
;; MSG SIZE  rcvd: 139
```
### 4. 備考
- スレーブへのゾーン転送は named.conf の allow-transfer で許可が必要
- スレーブサーバの /var/named/slaves/ にゾーンファイルが自動生成される
- serial の更新がないと転送されないためゾーンファイル変更時は必ず serial を増加
- フェイルオーバー確認時は systemctl stop named → dig で確認する
- キャッシュの影響を避けるため、テスト時は dig +trace や rndc flush が有効

### 補足
- dig +short を使うと結果だけを簡潔に表示できる
- host stream.lab.lan でも名前解決の確認が可能
- BINDログの監視には journalctl -u named -f が便利
