# マテビューログテーブル作成

マテビューログを作成すると下記３オブジェクトが生成される。
（設定によっては作成されないオブジェクトもあり）

| オブジェクト名 | タイプ | 説明 |
| --- | --- | --- |
| MLOG$_[テーブル名] | TABLE | 差分格納テーブル |
| I_MLOG$_[テーブル名] | INDEX | MLOG$_[テーブル名] テーブルに対するインデックス |
| RUPD$_[テーブル名] | GLOBAL TEMPORARY TABLE | 差分格納一時テーブル |


```SQL
-- マテビューログテーブル削除
DROP MATERIALIZED VIEW LOG ON [テーブル]

-- マテビューログテーブル作成
CREATE MATERIALIZED VIEW LOG ON [テーブル]
TABLESPACE [表領域]
NOLOGGING
WITH ROWID SEQUENCE
(
  [項目]
, [項目]
)
INCLUDING NEW VALUES
PURGE REPEAT INTERVAL '7' DAY
/

-- インデックスの表領域を移動
ALTER INDEX I_MLOG$_[テーブル名] REBUILD TABLESPACE [表領域]
/

-- マテビューログテーブル一覧
SELECT * FROM SYS.ALL_MVIEW_LOGS ORDER BY LOG_OWNER, MASTER
/
```

- 集計関数を使用していない場合  
WITH PRIMARY KEY ROWID  

- 集計関数を使用している場合（UPDATEでリフレッシュが必要な場合も対象）  
WITH ROWID, SEQUENCE (項目)  
INCLUDING NEW VALUES  
マテビュー内で参照している項目をすべて追加。但し、PRIMARY KEYの項目は追加不要（追加するとエラーになる）  

# マテビュー

- 内部結合、外部結合の書き方は、ORACLE独自の書き方で記述しないとマテビュー化できない


```SQL
-- マテビュー一覧
SELECT * FROM SYS.ALL_MVIEWS ORDER BY OWNER, MVIEW_NAME
/

-- リフレッシュ設定変更：リフレッシュグループを使用する場合
ALTER MATERIALIZED VIEW [マテビュー]
REFRESH FAST ON DEMAND
/

-- リフレッシュ設定変更：リフレッシュグループを使用しない場合
ALTER MATERIALIZED VIEW [マテビュー]
REFRESH FAST
START WITH ROUND(SYSDATE) + 11/24
NEXT SYSDATE + 1 / 24 / 60 * 20
/

-- マテビュー設定チェック
DECLARE
        msg_array SYS.ExplainMVArrayType;
BEGIN
        DBMS_MVIEW.EXPLAIN_MVIEW( 'マテビュー名', msg_array );
        DBMS_OUTPUT.ENABLE;

        FOR IDX IN 1..msg_array.COUNT LOOP

                DBMS_OUTPUT.PUT_LINE
                (
                        msg_array( IDX ).capability_name  || ' ' ||
                        msg_array( IDX ).possible         || ' ' ||
                        msg_array( IDX ).related_num      || ' ' ||
                        msg_array( IDX ).related_text     || ' ' ||
                        msg_array( IDX ).msgno            || ' ' ||
                        msg_array( IDX ).msgtxt
                );
        END LOOP;
END;
/
```

下記４項目の設定を確認
- REFRESH_FAST
- REFRESH_FAST_AFTER_INSERT
- REFRESH_FAST_AFTER_ONETAB_DML
- REFRESH_FAST_AFTER_ANY_DML

```SQL
-- リフレッシュグループ一覧
SELECT * FROM SYS.ALL_REFRESH ORDER BY ROWNER, RNAME
/
-- リフレッシュグループ内のオブジェクト一覧
SELECT * FROM SYS.ALL_REFRESH_CHILDREN ORDER BY OWNER, NAME
/
-- インスタンス内で現在実行中のジョブ
SELECT * FROM DBA_JOBS_RUNNING
/

-- リフレッシュグループ削除
BEGIN
DBMS_REFRESH.DESTROY( NAME => 'リフレッシュグループ' );
END;
/

-- リフレッシュグループ作成
BEGIN
DBMS_REFRESH.MAKE
( 
    NAME      => 'リフレッシュグループ' 
,   LIST      => 'マテビューの名前をカンマ区切りで指定'
,   NEXT_DATE => TO_DATE( '202501010101' 'YYYYMMDDHH24MI' )
,   INTERVAL  => 'SYSDATE + 1 / 24 / 60 * 20'
);
END;
/

-- リフレッシュ間隔の設定変更
BEGIN
DBMS_REFRESH.CHANGE
( 
    NAME      => 'リフレッシュグループ' 
,   NEXT_DATE => TO_DATE( '202501010101' 'YYYYMMDDHH24MI' )
,   INTERVAL  => 'SYSDATE + 1 / 24 / 60 * 20'
);
END;
/

-- リフレッシュグループ手動リフレッシュ
BEGIN
DBMS_REFRESH.REFRESH( NAME => 'リフレッシュグループ' );
END;
/

-- マテビュー手動リフレッシュ (f:高速、c:完全)
BEGIN
DBMS_MVIEW.REFRESH( LIST => 'マテビュー', METHOD => 'f' );
END;
/

```
