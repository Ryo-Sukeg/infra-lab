# NTP Server 構築・検証記録
本リポジトリでは、`chrony` を使用した NTP サーバ構築および検証の手順を記録しています。  
CentOS Stream / Ubuntu / AlmaLinux / RHEL のマルチ環境で動作確認を行いました。
---
## ファイル構成
| ファイル名 | 内容 |
|-------------|------|
| [NTP_Server_Setup.md](./NTP_Server_Setup.md) | 構築手順まとめ |
| [NTP_Server_Verification.md](./NTP_Server_Verification.md) | 検証結果まとめ |
---
## 環境概要
| 項目 | 内容 |
|------|------|
| OS構成 | CentOS Stream 9.6 / Ubuntu 24.04 / AlmaLinux 9.6 / RHEL 9.6 |
| 使用サービス | chrony |
| 構築目的 | NTPサーバの構築と冗長化検証 |
| 構成図 | master (CentOS) ⇔ slave (Ubuntu) ⇔ client (Alma/RHEL) |
---
## 学びポイント
- chrony は ntpd より軽量で仮想環境との相性が良い  
- フェイルオーバー時も自動で再同期し安定した時刻精度を維持  
- allow ディレクティブで許可範囲を限定しセキュリティ確保  
- 仮想環境での時刻ズレ対策として定期同期を推奨  