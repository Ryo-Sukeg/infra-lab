# NTP Server 検証結果記録

## 1. NTPサーバ検証結果 ( CentOS Stream 9 )

### `# chronyc sources`  
```text
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================  
  ^* ntp-b3.nict.go.jp             1   6    17     0    -62us[ -530us] +/- 4676us  
  ^- ntp2.jst.mfeed.ad.jp          2   6    17     1  +1156us[ +688us] +/-   29ms  
```
### `# chronyc tracking`  
```text
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
## 2. クライアント検証結果 ( RHEL 9.6 )  

### `# chronyc sources`  
```text
  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^* Stream                        2   6    17     7   -640us[ -577us] +/- 6066us  
  ^+ Ubuntu24                      2   6    17     7   +653us[ +716us] +/- 5845us 
```
### `# chronyc tracking`  
```text
  Reference ID    : C0A83866 (Ubuntu24)  
  Stratum         : 3  
  Ref time (UTC)  : Wed Oct 08 05:50:55 2025  
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
## 3. フェイルオーバーテスト ( RHEL 9.6 )  

### Ubuntu (192.168.56.102) chronyd 停止時  

### `# chronyc sources`  
```text
  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^+ Stream                        2   7   377    10   -986us[ -986us] +/- 6666us  
  ^* Ubuntu24                      2   7   377   204  +1697us[+1730us] +/- 8240us  
```
### CentOS_Stream (192.168.56.101) chronyd 停止時  

### `# chronyc sources`  
```text
  MS Name/IP address         Stratum Poll Reach LastRx Last sample  
  ===============================================================================  
  ^* Stream                        2   7   377    91   -596us[ -986us] +/- 6666us  
  ^+ Ubuntu24                      2   6   375    27    -58us[ -450us] +/-   10ms  
```
## 4. 検証まとめ  
- 両ノードが Stratum 2 の上位NTPに同期していることを確認  
- フェイルオーバー動作も正常でいずれかのサーバ停止時にも時刻同期を維持  
- chrony の `Reach` 値が 377 で安定しており通信状態も良好  
