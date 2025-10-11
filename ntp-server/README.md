# NTP Server 構築・検証記録 
本リポジトリでは `chrony` を使用した NTP サーバ構築および検証の手順を記録しています。  
CentOS Stream / Ubuntu / AlmaLinux / RHEL のマルチ環境で動作確認を行いました。

### ファイル構成
| ファイル名 | 内 容 |
|-------------|------|
| [NTP_Server_Setup.md](./NTP_Server_Setup.md) | 構築手順まとめ |
| [NTP_Server_Verification.md](./NTP_Server_Verification.md) | 検証結果まとめ |

### 環境概要
| 項 目 | 内 容 |
|------|------|
| OS構成 | CentOS Stream 9.6 / Ubuntu 24.04 / AlmaLinux 9.6 / RHEL 9.6 |
| 使用サービス | chrony |
| 構築目的 | NTPサーバの構築と冗長化の検証 |
| 構成図 | master (CentOS) ⇔ slave (Ubuntu) ⇔ client (Alma/RHEL) |

### 備考
- chrony は ntpd より軽量で仮想環境と相性が良い  
- フェイルオーバー時も自動で再同期し安定した時刻精度を維持  
- allow ディレクティブで許可範囲を限定してセキュリティ確保  
- 仮想環境での時刻ズレ対策として定期同期を推奨

 <img width="1491" height="787" alt="image" src="https://github.com/user-attachments/assets/4a59029c-80f7-42ba-9291-f9ee942e69bb" />
