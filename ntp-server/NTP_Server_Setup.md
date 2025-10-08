# NTP Server 構築手順記録

## 構成概要
| 項目 | 内容 |
|------|------|
| OS | master: CentOS\_Stream 9.6 / slave: Ubuntu 24.04 |
| サービス | chrony |
| 役割 | NTPサーバ（時刻同期） |
| クライアント | AlmaLinux 9.6 / RHEL 9.6 |
---

## 手順
### 1.NTPサーバ設定
```bash
# 1-1. chrony のインストール
Red Hat / CentOS 系
  sudo dnf install -y chrony
Debian / Ubuntu 系
  sudo apt-get install -y chrony

# 1-2. 設定ファイル編集
/etc/chrony.conf に以下を設定
  server ntp.nict.jp iburst
  server ntp.jst.mfeed.ad.jp iburst
  allow 192.168.56.0/24
※ Debian/Ubuntu系は /etc/chrony/chrony.conf

# 1-3. サービス起動と動作確認
  sudo systemctl enable --now chronyd
  sudo systemctl status chronyd

# 1-4. ファイアウォール許可設定と確認
  sudo firewall-cmd --permanent --add-service=ntp
  sudo firewall-cmd --reload
  sudo firewall-cmd --list-all
```
### 2. クライアント設定
```bash
# 2-1. chronyのインストール
  sudo dnf install -y chrony

# 2-2. 設定ファイル編集
 /etc/chrony.conf に以下を設定：
  server 192.168.56.101 iburst
  server 192.168.56.102 iburst

# 2-3. サービス起動
  sudo systemctl enable --now chronyd
  sudo systemctl status chronyd

# 2-4. ファイアウォール設定
  sudo firewall-cmd --permanent --add-service=ntp
  sudo firewall-cmd --reload
```
### 3. 学び・注意点
・NTPクライアントの許可範囲は /etc/chrony.conf の allow で指定  
・iburst オプションはサービス開始時の初期同期を高速化する  
・設定変更後は sudo systemctl restart chronyd でサービス再起動  
・NTPは UDP123番ポートを使用  
・仮想環境では時刻差が出やすいため同期確認を行う  
・`RMS offset` の履歴をリセットしたい場合は Chrony を再起動  

### 4. 気になったコマンド
```bash
# タイムゾーンの確認・変更
  timedatectl status
  sudo timedatectl set-timezone Asia/Tokyo

# 時刻確認・一時的手動同期
  date
  sudo chronyc makestep

# UDP/123番ポート開放確認 (簡易チェック)
  nc -uvz 192.168.56.101 123
```