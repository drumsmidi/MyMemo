
[Azure のドキュメント]
(https://learn.microsoft.com/ja-jp/azure/?product=popular)

--------------------------------------------------
# VM

[料金計算ツール](https://azure.microsoft.com/ja-jp/pricing/calculator/)

サーバー構成の検討に使用
（VMシリーズの決定やディスク性能の確認など）

--------------------------------------------------
# 負荷分散

## Azure Application Gateway

[Azure Application Gateway のドキュメント](https://learn.microsoft.com/ja-jp/azure/application-gateway/)

### セッションタイムアウトについて

FrontEnd側でプライベートIPを使用していると、4分か5分のセッションタイムアウト設定（固定）があり、パッケージ製品などでポーリング対応していないシステムで、長時間実行の処理がある場合、死ぬ

パブリックIPを使用すると30分まで伸ばせるが
外部公開されてしまうのでアクセス制限が必要となる

バックエンド側もセッションタイムアウト設定があるが
こちらは3000秒あたりまで伸ばせたはず

Apache や Tomcat のセッションタイムアウトに合わせて設定する

### TLS/SSL証明書登録が必要

エンド ツー エンド TLS 暗号化とする場合 
APサーバー、ApplicationGateway側両方に証明書の設定が必要。

証明書は同じものでも異なっていても大丈夫な模様。

cookie ベースのセッション アフィニティを使用する。
これにて利用者ごとにバックエンドの振り分け先が決定するはず。


## Azure Load Balancer

[Load Balancer のドキュメント](https://learn.microsoft.com/ja-jp/azure/load-balancer/)

VMサーバーへのFTP接続などのアクセス振り分けやJP1などのジョブ実行振り分け等で使用できる

### セッションタイムアウトについて

確か無期限設定ができたので処理が途中で中断されることはない。ジョブなど長時間実行があっても問題なし

--------------------------------------------------
# Azure ExpressRoute

オンプレミス環境とAzure環境間のネットワーク接続を確率する仕組み。

オンプレミス環境⇔Azure環境間の通信は遅いので注意が必要。
回線の帯域幅のオプションがあるようで、オンプレからAzureへデータ移行する際には注意が必要。

NSGの設定が必要となる。
| 接続元 | 接続先 | NSG要否 |
| --- | --- | --- |
| オンプレ | Azure | 不要 |
| Azure | 外部インターネット | 不要 |
| Azure | オンプレ | 要 |
| Azure(Internal) | Azure(External) | 要 |
| Azure(External) | Azure(Internal) | 要 |

> Azureの配置サブネット違いやVNET統合したAppServiceからの接続についても申請が必要

> AppServiceについては、基本パブリック公開となっているが
> ExpressRoute？を専用回線で引いている場合
> 専用回線内でのパブリック公開的な話を聞いたことがある
>
> ただ、アクセス元の制限はかけた方が安全

--------------------------------------------------
# Oracle Database@Azure

[Oracle Database@Azure](https://www.oracle.com/jp/cloud/azure/oracle-database-at-azure/)

Azure環境でOCI相当のサービス提供が開始された。
25年4月時点で、リージョン：東日本でサービス提供開始されている。
（西日本は未定）

このサービスは、Exadata上のOCIベースのOracle Autonomous Database を使用することになる。
Exadata上なので恐らく早い。

Oracle Autonomous Databaseは、AI導入による運用の自動化敵なものらしい。Oracle19c以降であれば、互換性は問題なさそう？

PaaSサービスになるので、UTL_FILE系の処理は何か対応が必要。
Azure BloBサービスを通してやり取りはできそう。
ただ、そうするぐらいならアプリ側でファイル入出力をカバーした方がいいのでは？と思う。

[Oracle データベースを Exadata Database Service OD@Aに移行する](https://learn.microsoft.com/ja-jp/azure/architecture/databases/idea/migrate-oracle-odaa-exadata)

--------------------------------------------------
# Azure Backup

## VM

LRS：同じリージョン内に、同期的に３つコピーされる  
GRS：LRSの処理後、別リージョンに非同期でレプリケートする  

本番のみ または DB関連のサーバーのみ GRS にしておいた方がいいのかな？


