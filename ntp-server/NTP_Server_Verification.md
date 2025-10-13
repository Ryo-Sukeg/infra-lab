# NTP Server 検証結果記録

## 構成概要

| サーバ種別 | O S | ホスト名 | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| NTP | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | 時刻同期 |
| NTP | Ubuntu 24.04.3 | ubuntu.lab.lan | 192.168.56.102 | 時刻同期 |
| client | RHEL 9.6 | rhel.lab.lan | 192.168.56.103 |      - |
| client | AlmaLinux 9.6 | alma.lab.lan| 192.168.56.104 | フェイルオーバーテスト |
---
### 1. NTPサーバの状態確認・検証結果（CentOS Stream 9）
```
sudo systemctl status chronyd
chronyc sources
chronyc tracking
```
出力詳細：
```
$ sudo systemctl status chronyd

● chronyd.service - NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-10-08 16:54:51 JST; 26min ago
       Docs: man:chronyd(8)
             man:chrony.conf(5)
    Process: 706 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 714 (chronyd)
      Tasks: 1 (limit: 10651)
     Memory: 4.6M (peak: 5.1M)
        CPU: 177ms
     CGroup: /system.slice/chronyd.service
             mq714 /usr/sbin/chronyd -F 2

10月 08 16:54:51 Stream9.6 chronyd[714]: chronyd version 4.6.1 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP>
10月 08 16:54:51 Stream9.6 chronyd[714]: Loaded 0 symmetric keys
10月 08 16:54:51 Stream9.6 chronyd[714]: Using right/UTC timezone to obtain leap second data
10月 08 16:54:51 Stream9.6 chronyd[714]: Frequency -21.464 +/- 3.041 ppm read from /var/lib/chrony/drift
10月 08 16:54:51 Stream9.6 chronyd[714]: Loaded seccomp filter (level 2)
10月 08 16:54:51 Stream9.6 systemd[1]: Started NTP client/server.
10月 08 16:55:28 Stream9.6 chronyd[714]: Selected source 133.243.238.163 (ntp.nict.jp)
10月 08 16:55:28 Stream9.6 chronyd[714]: System clock wrong by 1.876661 seconds
10月 08 16:55:30 Stream9.6 chronyd[714]: System clock was stepped by 1.876661 seconds
10月 08 16:55:30 Stream9.6 chronyd[714]: System clock TAI offset set to 37 seconds
```
```
$ chronyc sources

  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================  
  ^* ntp-b3.nict.go.jp             1   6    17     0    -62us[ -530us] +/- 4676us  
  ^- ntp2.jst.mfeed.ad.jp          2   6    17     1  +1156us[ +688us] +/-   29ms  
```
```
$ chronyc tracking

  Reference ID    : 85F3EEA4 (ntp-b3.nict.go.jp)  
  Stratum         : 2  
  Ref time (UTC)  : Wed Oct 08 05:45:27 2025  
  System time     : 0.000057039 seconds slow of NTP time  
  Last offset     : -0.000057812 seconds  
  RMS offset      : 0.001101404 seconds  
  Frequency       : 21.656 ppm slow  
  Residual freq   : -0.698 ppm  
  Skew            : 0.500 ppm  
  Root delay      : 0.010833539 seconds  
  Root dispersion : 0.000397551 seconds  
  Update interval : 64.7 seconds  
  Leap status     : Normal  
```
### 2. クライアントの状態確認・検証結果（RHEL 9.6）  
```
sudo systemctl status chronyd
chronyc sources
chronyc tracking
```
出力詳細：
```
$ sudo systemctl status chronyd

● chronyd.service - NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-10-08 17:54:38 JST; 1min 14s ago
       Docs: man:chronyd(8)
             man:chrony.conf(5)
    Process: 717 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
   Main PID: 719 (chronyd)
      Tasks: 1 (limit: 11066)
     Memory: 4.3M
        CPU: 246ms
     CGroup: /system.slice/chronyd.service
             mq719 /usr/sbin/chronyd -F 2

10月 08 17:54:38 RHEL9.6 systemd[1]: Started NTP client/server.
10月 08 17:54:41 RHEL9.6 chronyd[719]: Source 192.168.56.102 offline
10月 08 17:54:41 RHEL9.6 chronyd[719]: Source 192.168.56.101 offline
10月 08 17:54:43 RHEL9.6 chronyd[719]: Source 192.168.56.102 online
10月 08 17:54:43 RHEL9.6 chronyd[719]: Source 192.168.56.101 online
10月 08 17:54:48 RHEL9.6 chronyd[719]: Selected source 192.168.56.101
10月 08 17:54:48 RHEL9.6 chronyd[719]: System clock wrong by 1.784513 seconds
10月 08 17:54:50 RHEL9.6 chronyd[719]: System clock was stepped by 1.784513 seconds
10月 08 17:54:50 RHEL9.6 chronyd[719]: System clock TAI offset set to 37 seconds
10月 08 17:54:52 RHEL9.6 chronyd[719]: Selected source 192.168.56.102
```
```
$ chronyc sources

  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^* Stream                        2   6    17     7   -640us[ -577us] +/- 6066us  
  ^+ Ubuntu24                      2   6    17     7   +653us[ +716us] +/- 5845us 
```
```
$ chronyc tracking

  Reference ID    : C0A83866 (Ubuntu24)  
  Stratum         : 3  
  Ref time (UTC)  : Wed Oct 08 08:58:06 2025  
  System time     : 0.000220787 seconds slow of NTP time  
  Last offset     : +0.000098491 seconds  
  RMS offset      : 0.000740090 seconds  
  Frequency       : 21.705 ppm slow  
  Residual freq   : +0.040 ppm  
  Skew            : 2.497 ppm  
  Root delay      : 0.009981629 seconds  
  Root dispersion : 0.001442446 seconds  
  Update interval : 64.8 seconds  
  Leap status     : Normal  
```
### 3. フェイルオーバーテスト（RHEL 9.6）  

Ubuntu（192.168.56.102）chronyd 停止時  
```
$ chronyc sources

  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^+ Stream                        2   7   377    10   -986us[ -986us] +/- 6666us  
  ^* Ubuntu24                      2   7   377   204  +1697us[+1730us] +/- 8240us  
```
CentOS_Stream（192.168.56.101）chronyd 停止時  
```
$ chronyc sources

  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^* Stream                        2   7   377    91   -596us[ -986us] +/- 6666us  
  ^+ Ubuntu24                      2   6   375    27    -58us[ -450us] +/-   10ms  
```
### 4. 検証まとめ  
- 両ノードが Stratum 2 の上位NTPに同期していることを確認  
- フェイルオーバー動作も正常でいずれかのサーバ停止時にも時刻同期を維持  
- chrony の `Reach` 値が 377 で安定しており通信状態も良好  
