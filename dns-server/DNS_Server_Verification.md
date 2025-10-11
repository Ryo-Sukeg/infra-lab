# DNS Server 検証結果記録 (lab.lan)

## 構成概要

| サーバ種別 | OS / ホスト名 | IPアドレス | 役割 |
|-------------|----------------|-------------|------|
| master | CentOS Stream 9 / stream.lab.lan | 192.168.56.101 | DNS マスターサーバ |
| slave | Ubuntu 24.04 / ubuntu.lab.lan | 192.168.56.102 | DNS スレーブサーバ |
| client | RHEL 9.6 / rhel.lab.lan | 192.168.56.103 | 正引き・逆引き確認 |
| client | AlmaLinux 9.6 / alma.lab.lan| 192.168.56.104 | フェイルオーバーテスト |

---

### 1. マスター・スレーブの状態確認

stream.lab.lan (master DNS)

```bash
sudo systemctl status named
sudo rndc status
```
出力例：
```
version: 9.18.24-RedHat-9.18.24-1.el9_4.1
number of zones: 3
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is OFF
server is up and running
```
ubuntu.lab.lan (slave DNS)
```
sudo systemctl status named
sudo tail -n 10 /var/log/syslog
```
出力例：
```
zone lab.lan/IN: Transfer started.
transfer of 'lab.lan/IN' from 192.168.56.101#53: Transfer completed: 1 messages, 12 records, 328 bytes, 0.002 secs (164000 bytes/sec)
zone lab.lan/IN: loaded serial 2025100801
zone lab.lan/IN: Transfer completed successfully.
```
### 2. 正引き・逆引き確認
クライアント（RHEL9.6）
```
dig stream.lab.lan
dig ubuntu.lab.lan
dig -x 192.168.56.101
```
出力例：
```
;; ANSWER SECTION:
stream.lab.lan.      3600    IN    A    192.168.56.101
ubuntu.lab.lan.       3600    IN    A    192.168.56.102

;; ANSWER SECTION:
101.56.168.192.in-addr.arpa.  3600  IN  PTR  stream.lab.lan.
```
### 3. フェイルオーバーテスト
マスター停止時（スレーブ応答確認）
```
# master側で停止
sudo systemctl stop named

# クライアントから確認
dig @192.168.56.102 stream.lab.lan
```
出力例：
```
;; SERVER: 192.168.56.102#53(ubuntu.lab.lan)
;; ANSWER SECTION:
stream.lab.lan.    3600  IN  A  192.168.56.101
```
スレーブ停止時（マスター応答確認）
```
# slave側で停止
sudo systemctl stop named

# クライアントから確認
dig @192.168.56.101 ubuntu.lab.lan
```
出力例：
```
;; SERVER: 192.168.56.101#53(stream.lab.lan)
;; ANSWER SECTION:
ubuntu.lab.lan.     3600  IN  A  192.168.56.102
```
### 4. 備考
- スレーブへのゾーン転送は named.conf の allow-transfer で許可が必要
- スレーブサーバの /var/named/slaves/ にゾーンファイルが自動生成される
- serial の更新がないと転送されないためゾーンファイル変更時は必ず serial を増加
- フェイルオーバー確認時は systemctl stop named → dig で確認する
- キャッシュの影響を避けるため、テスト時は dig +trace や rndc flush が有効

補足
- dig +short を使うと結果だけを簡潔に表示できる
- host stream.lab.lan でも名前解決の確認が可能
- BINDログの監視には journalctl -u named -f が便利