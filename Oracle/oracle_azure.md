# Oracle Database@Azure

Oracle Database@Azure
https://www.oracle.com/jp/cloud/azure/oracle-database-at-azure/

25年4月時点では、東日本は提供開始されている。  
西日本はまだ未定。  

ファイル入出力系の処理を使用していると影響を受ける。  
- Azure File をNFSマウントして対応ができるかも？  
- Azure Blob を使用してファイル入出力はできそう。ただ、認証、連携と手間が増えそう  

## Cloud Premigration Advisor Tool

Oracle DatabaseからAutonomous Databaseへ移行する際の互換性チェックに使用。  

参考
https://oracle-japan.github.io/ocitutorials/adb/adb302-cpat/

JRE1.8以降がインストールされている環境であれば  
Windows/Linuxどちらでも実行可能。  

Cloud Premigration Advisor Tool をダウンロードするには  
OracleのサポートIDが必要となる。

Autonomous Databaseの場合、移行先の環境を指定する必要がある。  

デプロイメント・タイプ
- Dedicated：ATPD or ADWD
- Shared：ATPS or ADWS

ChatGptに聞いてみた

共有のサーバーを使用するか、サーバー単位で使用するかの選択っぽい  
| 項目 | Shared Infrastructure（共有） | Dedicated Infrastructure（専用） |
| --- | --- | --- |
| インフラ | 他ユーザーと共有 | 専用（Exadata単位） |
| 起動時間 | 即時 | 数分（起動・停止が可能） |
| 管理レベル | Oracleが全自動 | より細かく設定可能（管理者向け） |
| マルチテナント（PDB） | 単一PDBのみ（1データベース） | 複数PDB作成可能（CDB構成） |
| セキュリティ | 通常のセキュリティ | より厳密な分離と制御が可能 |
| ユースケース | スタンダードなクラウド利用 | 金融、医療、大企業など厳格要件あり |

基本的には、トランザクション処理なのかなと思う  
| タイプ | 用途 | 特徴 |
| --- | --- | --- |
| ADW | データ分析 | 並列処理、BI連携最適化 |
| ATP | トランザクション処理 | 低レイテンシ、OLTP向け |
| AJD | JSONアプリ向け | NoSQL的用途に最適 |
| Dedicated / Cloud@Customer | 配置環境の違い | セキュリティや構成の柔軟性 |

| シナリオ | おすすめ |
| --- | --- |
| 中小規模アプリ、迅速に使いたい | ATP (Shared) または ADW (Shared) |
| 分析・BI用データ基盤を構築 | ADW (Shared) または ADW (Dedicated) |
| 企業システム、複数DBを一括管理したい | ATP/ADW (Dedicated) |
| セキュリティポリシーが厳しい業界 | Dedicated Infrastructure 一択 |

| 状況 | 選ぶべきもの |
| --- | --- |
| スピーディーに作ってコストも抑えたい | ATPS / ADWS（Shared） |
| より細かく設定・分離したい（マルチPDBなど） | ATPD / ADWD（Dedicated） |
| 分析ワークロードで高速パフォーマンスを活用 | ADWS または ADWD |
| 高頻度OLTPでトランザクションが多い | ATPS または ATPD |

