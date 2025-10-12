# DNS Server 検証結果記録 (lab.lan)

## 構成概要

| サーバ種別 | O S | ホスト名 | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| DNS | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | マスターサーバ |
| DNS | Ubuntu 24.04 | ubuntu.lab.lan | 192.168.56.102 | スレーブサーバ |
| client | RHEL 9.6 | rhel.lab.lan | 192.168.56.103 | 正引き・逆引き確認 |
| client | AlmaLinux 9.6 | alma.lab.lan| 192.168.56.104 | フェイルオーバーテスト |

---

### 1. マスター・スレーブの状態確認

stream.lab.lan ( master DNS )

```bash
sudo systemctl status named
sudo rndc status
```
出力例：
```
rndc status
version: BIND 9.16.23-RH (Extended Support Version) <id:fde3b1f>
running on Stream9.6: Linux x86_64 5.14.0-620.el9.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Sep 26 01:13:23 UTC 2025
boot time: Sat, 11 Oct 2025 03:43:56 GMT
last configured: Sat, 11 Oct 2025 03:43:57 GMT
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
ubuntu.lab.lan ( slave DNS )
```
sudo systemctl status named
sudo tail -n 5 /var/log/syslog
```
出力例：
```
2025-10-11T12:54:43.410743+09:00 Ubuntu24 rtkit-daemon[1536]: Supervising 8 threads of 5 processes of 1 users.
2025-10-11T12:54:52.820420+09:00 Ubuntu24 PackageKit: daemon quit
2025-10-11T12:54:52.842837+09:00 Ubuntu24 systemd[1]: packagekit.service: Deactivated successfully.
2025-10-11T12:55:01.373844+09:00 Ubuntu24 CRON[2306]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
2025-10-11T12:55:21.147014+09:00 Ubuntu24 systemd[1502]: launchpadlib-cache-clean.service - Clean up old files in the Launchpadlib cache was skipped because of an unmet condition check (ConditionPathExists=/var/lib/gdm3/.launchpadlib/api.launchpad.net/cache).
```
### 2. 正引き・逆引き確認
クライアント（ RHEL9.6 ）
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
