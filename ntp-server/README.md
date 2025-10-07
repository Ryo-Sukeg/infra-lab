# NTP Server 構築記録

## 構成概要
| 項目 | 内容 |
|------|------|
| OS | AlmaLinux 9.6 |
| サービス | chrony |
| 役割 | NTPサーバ（時刻同期） |
| クライアント | Ubuntu 24.04 / RHEL 9.6 |

---

## 手順

### 1. chronyのインストール
```bash
sudo dnf install -y chrony

### 2. 設定ファイル編集
/etc/chrony.conf に以下を設定：
allow 192.168.56.0/24
local stratum 10

### 3. サービス起動
sudo systemctl enable --now chronyd
sudo systemctl status chronyd

### 4. ファイアウォール許可
sudo firewall-cmd --permanent --add-service=ntp
sudo firewall-cmd --reload

### 検証結果
chronyc sources -v
210 Number of sources = 2
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ntp.nict.jp                   1   6   377    34  +123us[ +234us] +/-  2ms

学び・注意点
・NTPクライアントの許可範囲は /etc/chrony.conf の allow で指定。
・仮想環境では時刻差が出やすいので、同期確認を習慣化する。