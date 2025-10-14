# NTP Server 構築・検証記録 
本リポジトリは `chrony` を使用した NTP サーバの構築手順と検証結果を記録しています。NTPサーバは2台構築し冗長化の確認も行っています。

### ファイル構成
| ファイル名 | 内 容 |
|-------------|------|
| [NTP_Server_Setup.md](./NTP_Server_Setup.md) | 構築手順まとめ |
| [NTP_Server_Verification.md](./NTP_Server_Verification.md) | 検証結果まとめ |

### 構成情報  
| 役 割 | ホスト名 | O S | IPアドレス | 主なサービス |  
|------|-----------|----|-------------|---------------|  
| NTPサーバ | Stream | CentOS Stream 9 | 192.168.56.101 | chrony / BIND (named) |  
| NTPサーバ | Ubuntu24 | Ubuntu 24.04.3 | 192.168.56.102 | chrony / BIND (named) |  
| クライアント | rhel9.6 | RHEL 9.6 | 192.168.56.103 | samba / NFS |  
| クライアント | alma9.6 | AlmaLinux 9.6 | 192.168.56.104 | LAMP環境 / Zabbix | 

### 備考
- chrony は ntpd より軽量で仮想環境と相性が良い  
- フェイルオーバー時も自動で再同期し安定した時刻精度を維持  
- allow ディレクティブで許可範囲を限定してセキュリティ確保  
- 仮想環境での時刻ズレ対策として定期同期を推奨

 <img width="1491" height="787" alt="image" src="https://github.com/user-attachments/assets/4a59029c-80f7-42ba-9291-f9ee942e69bb" />
