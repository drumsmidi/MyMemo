参考：https://qiita.com/nullnull/items/a5c12e73c6246d6f97fe



# 開発環境

Salesforce 開発のための Visual Studio Code(VSCode)  
https://trailhead.salesforce.com/ja/content/learn/projects/quickstart-vscode-salesforce/use-vscode-for-salesforce



- JDK 11以降？
- SFDX(SalesForce DX) CLI
- Salesforce extension pack


- VSCode
  - Japanese Language Pack for Visual Studio Code  
  - [Salesforce Extension Pack](https://developer.salesforce.com/docs/platform/sfvscode-extensions/guide)  



# DB

[参考：Salesforceのデータベースとは？概要やその仕組を徹底解説！](https://www.ccc.seraku.co.jp/service/columns/salesforce/02/125/)

Salesforceの場合、行を「レコード」、列を「フィールド」、テーブルを「オブジェクト」と呼ぶそう。


標準オブジェクト
　アカウントオブジェクト
　リードオブジェクト
　コンタクトオブジェクト

カスタムオブジェクト
　ユーザの独自のニーズにより作成されるオブジェクト
　「ID」「ネーム」「システム」の標準フィールドが自動的に付属する
　　ID：各レコードに対する一意のデータ識別子（15文字）
　　システム：通常、そのレコードが最後に更新された日付と時間を含む文字列（Last-Modified）

外部オブジェクト
　Salesforceの外部にあるデータをマッピングして読み込むカスタムオブジェクト


参照関係（ルックアップリレーションシップ）
　検索キー項目

主従関係（マスター詳細リレーションシップ）
　主のレコードを削除すると従のレコードも一緒に削除される関係


「CData Sync」を使用すると Oracle Database へデータ連携が可能
SalesForceを廃止する場合は、データ同期してから移行先のテーブルへ移行かな


