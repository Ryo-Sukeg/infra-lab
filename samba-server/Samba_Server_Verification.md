# Samba Server 検証結果（RHEL9.6）

## 構成概要

| サーバ種別 | O S | ホスト名 | IPアドレス | 役 割 |
|-------------|-----|-----------|-------------|------|
| Linux client | CentOS Stream 9 | stream.lab.lan | 192.168.56.101 | - |
| Linux client | Ubuntu 24.04.3 | ubuntu.lab.lan | 192.168.56.102 | - |
| Samba / NFS | RHEL 9.6 | rhel.lab.lan | 192.168.56.103 | ファイル共有 |
| Linux client | AlmaLinux 9.6 | alma.lab.lan| 192.168.56.104 | - |


## 1. 接続確認ログ

### Linux クライアント（Ubuntu 24.04）
```
smbclient -L //192.168.56.103 -U sambauser
Enter SAMBAUSER's password: 

Sharename       Type      Comment
---------       ----      -------
share           Disk      

sudo mount -t cifs //192.168.56.103/share /mnt/samba -o username=sambauser,password=SambaPass123,vers=3.0

ls /mnt/samba

test.txt

Windows クライアント

エクスプローラに \\192.168.56.103\share を入力

認証成功 → ファイル一覧が表示

test.txt の内容を確認・編集可能

### 2. 結果概要  
テスト項目	結果	備考
Samba 接続（Windows）		認証成功・ファイル操作可
Samba 接続（Linux CIFS）		vers=3.0 で接続確認
NFS 接続（Linux）		同一ディレクトリ同期確認
ファイル整合性 	Samba/NFS間で一致
不要共有非表示		[homes] 等削除済

### 3. 今後の課題  
アクセス権限をユーザー単位で制御
認証方式（NTLMv2）のセキュア化
ログローテーション設定
