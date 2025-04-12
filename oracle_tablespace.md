# 表領域チェック

```SQL
SELECT
        TABLESPACE_NAME
,       TO_CHAR( "SIZE(MB)"     , '9,999,999' ) "SIZE(MB)"
,       TO_CHAR( "USED(MB)"     , '9,999,999' ) "USED(MB)"
,       TO_CHAR( "FREE(MB)"     , '9,999,999' ) "FREE(MB)"
,       TO_CHAR( "MAX_SIZE(MB)" , '9,999,999' ) "MAX_SIZE(MB)"
,       TO_CHAR( "USE_RATE(%)"  , '990.00'    ) "USE_RATE(%)"
,       TO_CHAR( "MAX_RATE(%)"  , '990.00'    ) "MAX_RATE(%)"
,       CASE
                WHEN "MAX_RATE(%)" > 98 THEN '■■■警告'
                WHEN "MAX_RATE(%)" > 94 THEN '■■警告'
                WHEN "MAX_RATE(%)" > 90 THEN '□注意'
        END "CHK"
FROM
        (
                SELECT
                        A.TABLESPACE_NAME
                ,       A.TOTAL_BYTES                                    / 1024 / 1024                            "SIZE(MB)"
                ,       ( A.TOTAL_BYTES - NVL( B.FREE_TOTAL_BYTES, 0 ) ) / 1024 / 1024                            "USED(MB)"
                ,       NVL( B.FREE_TOTAL_BYTES, 0 )                     / 1024 / 1024                            "FREE(MB)"
                ,       A.TOTAL_MAXBYTES                                 / 1024 / 1024                            "MAX_SIZE(MB)"
                ,       DECODE( A.TOTAL_BYTES   , 0, 0, ROUND( ( A.TOTAL_BYTES - NVL( B.FREE_TOTAL_BYTES, 0 ) ) / A.TOTAL_BYTES    * 100, 2 ) )     "USE_RATE(%)"
                ,       DECODE( A.TOTAL_MAXBYTES, 0, 0, ROUND( ( A.TOTAL_BYTES - NVL( B.FREE_TOTAL_BYTES, 0 ) ) / A.TOTAL_MAXBYTES * 100, 2 ) )     "MAX_RATE(%)"
                FROM
                        (
                                SELECT
                                        TABLESPACE_NAME
                                ,       SUM( NVL( BYTES   , 0 ) ) "TOTAL_BYTES"
                                ,       SUM( NVL( MAXBYTES, 0 ) ) "TOTAL_MAXBYTES"
                                FROM
                                        DBA_DATA_FILES
                                GROUP BY
                                        TABLESPACE_NAME
                        ) A
                ,       (
                                SELECT
                                        TABLESPACE_NAME
                                ,       SUM( NVL( BYTES, 0 ) ) "FREE_TOTAL_BYTES"
                                FROM
                                        DBA_FREE_SPACE
                                GROUP BY
                                        TABLESPACE_NAME
                        ) B
                WHERE
                        A.TABLESPACE_NAME = B.TABLESPACE_NAME(+)
        ) A
ORDER BY
        "MAX_RATE(%)" DESC
/

-- 表領域のブロックチェック
SELECT
    E.SEGMENT_NAME
,   E.TABLESPACE_NAME
,   E.EXTENT_ID
,   E.BLOCK_ID
,   E.BLOCKS
FROM
    DBA_EXTENTS E
,   DBA_DATA_FILES  F
WHERE
    E.FILE_ID   = F.FILE_ID
--AND   E.SEGMENT_TYPE  = 'TABLE'
--AND   E.OWNER     = ''
AND F.FILE_NAME = 'F:\APP\ORADATA\DBNAME\TS_TBL.DBF'
ORDER BY
    E.BLOCK_ID DESC
/
```

# テーブルスペース圧縮 ONLINE状態で実行可能（効果：低）

```SQL
-- 行移動を有効にする
ALTER TABLE TEST.TBL_TEST ENABLE ROW MOVEMENT;
-- テーブルを圧縮する
ALTER TABLE TEST.TBL_TEST SHRINK SPACE CASCADE;
-- 行移動を無効にする
ALTER TABLE TEST.TBL_TEST DISABLE ROW MOVEMENT;
-- テーブルスペース移動、インデックスがあるテーブルの場合は注意が必要（特にプライマリーキーは注意）
ALTER TABLE TBL_TEST MOVE TABLESPACE TS_TBL2;

-- インデックス再構築
ALTER INDEX IDX1_TBL_TEST REBUILD;
ALTER INDEX IDX2_TBL_TEST REBUILD;
-- 一時表領域の縮小
ALTER TABLESPACE TEMP_TEST SHRINK SPACE KEEP 4G;
-- 表領域の書き込み可/不可の変更
-- 読取り専用表領域のデータにアクセスする際のパフォーマンスを向上させるため、表領域を読取り専用にする直前に
-- 表領域内の表のブロックすべてにアクセスする問合せを発行することをお薦めします。
-- 各表に対してSELECT COUNT (*)などの単純な問合せを実行しておくと
-- それ以降、表領域のデータ・ブロックに最も効率的にアクセスできるようになります。
-- これによって、最後にブロックを変更したトランザクションの状態をデータベースが確認する必要がなくなるからです。
SELECT COUNT(*) FROM <テーブル名>;
ALTER TABLESPACE <表領域名> READ ONLY;
ALTER TABLESPACE <表領域名> READ WRITE;
```

# 一時表領域の再作成

```SQL
-- 【手順1】 空きのあるディスク(フォルダ)に一時表領域を作成する
ALTER TABLESPACE TEMP_EBOM ADD TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST_AFTER_DELETE.DBF' SIZE 128M AUTOEXTEND OFF

-- 【手順2】 ディスク圧迫している一時表領域のデータファイルを削除するためオフラインにする
ALTER DATABASE TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST.DBF' OFFLINE
--ALTER DATABASE TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST.DBF' ONLINE

-- 【手順3】 手順2でオフラインにしたデータファイルを削除する
ALTER DATABASE TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST.DBF' DROP INCLUDING DATAFILES

-- 削除できない場合はアプリが使用している可能性があるので下記SQLでプロセスを確認し
-- 対象のプロセスを終了させてから再度削除を試みる。
SELECT
    S.USERNAME
,   S.OSUSER
,   S.MACHINE
,   S.TERMINAL
,   S.PROGRAM
FROM
    SYS.V_$SESSION S
,   SYS.V_$PROCESS P
WHERE
    S.PADDR = P.ADDR
/

-- 【手順4】 削除した一時表領域のデータファイルを再作成する
ALTER TABLESPACE TEMP_TEST ADD TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST.DBF' SIZE 2048M AUTOEXTEND OFF

-- 【手順5】 手順1で作成したデータファイルを削除するためオフラインにする
ALTER DATABASE TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST_AFTER_DELETE.DBF' OFFLINE

-- 【手順6】 手順5でオフラインにしたデータファイルを削除する
ALTER DATABASE TEMPFILE 'E:\APP\ORADATA\DBNAME\TEMP_TEST_AFTER_DELETE.DBF' DROP INCLUDING DATAFILES
```

# データファイルの移動

```SQL
-- 表領域をオフラインにする
ALTER TABLESPACE TS_TBL OFFLINE;

-- データファイルを移動先へコピーする
OSのコマンドなどでデータファイルをコピーします。

-- データファイルを変更する
ALTER TABLESPACE TS_TBL RENAME DATAFILE 'F:\APP\ORADATA\DBNAME\TS_TBL.ORA' TO 'E:\APP\ORADATA\DBNAME\TS_TBL.ORA';

-- 表領域をオンラインにする
ALTER TABLESPACE TS_TBL ONLINE;
```

# 圧縮表の作成
ディスク容量が圧迫してきたら検討

```SQL
-- テーブルスペースをデフォルトで圧縮表とする
-- （うろ覚えだが、テーブルスペースに対しては設定出来なかったかも）
CREATE TABLESPACE <表領域名>
DATAFILE '........'
DEFAULT COMPRESS FOR OLTP
/

-- 新規テーブル作成時に圧縮表のオプションを指定
CREATE TABLE <テーブル名> (column1,column2,..)
COMPRESS FOR OLTP
/

-- 既存テーブルを圧縮表に変更する場合
--   新規レコードだけではなく、既存レコードも圧縮する場合
--   索引のRebuildが必要
ALTER TABLE TBL_TEST MOVE TABLESPACE TS_TBL_TEMP;
ALTER TABLE TBL_TEST MOVE TABLESPACE TS_TBL COMPRESS FOR BASIC;
ALTER INDEX IDX1_TBL_TEST REBUILD;
ALTER INDEX IDX2_TBL_TEST REBUILD;

SELECT
    TABLE_NAME
,   COMPRESSION
,   COMPRESS_FOR
FROM
    USER_TABLES
WHERE
    COMPRESSION = 'ENABLED'
ORDER BY
    TABLE_NAME
/

SELECT
    TABLE_OWNER
,   TABLE_NAME
,   PARTITION_NAME
,   COMPRESSION
,   COMPRESS_FOR
FROM
    DBA_TAB_PARTITIONS
WHERE
    COMPRESSION = 'ENABLED'
ORDER BY
    TABLE_OWNER
,   TABLE_NAME
,   PARTITION_NAME
/
```
