# 非パーティション表をパーティション表へ変換

- パーティション化のキー項目は、NOT NULL 制約が必要。
- データがある状態でパーティション化を実施すると時間がかかるので、データが空の状態でパーティション化してからデータをインポートした方がよい
- ローカルインデックス、グローバルインデックスの違いは理解しておく

LISTパーティション表
```SQL
SET SERVEROUTPUT ON
DECLARE
    WK_SQL_TEXT1    VARCHAR2(30000);
    WK_SQL_TEXT2    VARCHAR2(30000);
    C_KEY           VARCHAR2(30)    := 'パーティション化キー項目';
    C_CRLF          VARCHAR2(2)     := CHR(10);
--  C_CRLF          VARCHAR2(2)     := CHR(13) || CHR(10);
BEGIN
    FOR REC_TBL IN
    (
        SELECT
            A.OWNER
        ,   A.TABLE_NAME        "TBL_NM"
        ,   A.TABLESPACE_NAME   "TBL_SPC_NM"
        FROM
            SYS.ALL_TABLES      A
        ,   ( SELECT OWNER, TABLE_NAME FROM SYS.ALL_TAB_COLUMNS WHERE COLUMN_NAME = C_KEY GROUP BY OWNER, TABLE_NAME ) B
        WHERE
            A.OWNER                     IN ('対象スキーマ－')
        AND A.TABLE_NAME                IN ('対象テーブル')
        AND A.PARTITIONED               = 'NO'
        AND A.OWNER                     = B.OWNER(+)
        AND A.TABLE_NAME                = B.TABLE_NAME(+)
        AND NVL2( B.OWNER, '〇', '■' ) = '〇'      -- パーティション化キーを含むテーブルを対象とする
        ORDER BY
            A.OWNER
        ,   A.TABLE_NAME
    )
    LOOP
        WK_SQL_TEXT2 := '';
        
        FOR REC_IDX IN
        (
            SELECT
                A.INDEX_NAME            "IDX_NM"
            ,   A.TABLESPACE_NAME       "IDX_SPC_NM"
            ,   CASE WHEN B.INDEX_OWNER IS NOT NULL THEN 'LOCAL' THEN 'GLOBAL' END "IDX_TYPE"
            FROM
                SYS.ALL_INDEXES     A
            ,   ( SELECT INDEX_OWNER, INDEX_NAME FROM SYS.ALL_IND_COLUMNS WHERE COLUMN_NAME = C_KEY GROUP BY INDEX_OWNER, INDEX_NAME ) B
            WHERE
                A.OWNER         = REC_TBL.OWNER
            AND A.TABLE_NAME    = REC_TBL.TBL_NM
            AND A.OWNER         = B.INDEX_OWNER(+)
            AND A.INDEX_NAME    = B.INDEX_NAME(+)
            ORDER BY
                A.OWNER
            ,   A.INDEX_NAME
        )
        LOOP
        
            IF TRIM( WK_SQL_TEXT2 ) IS NULL THEN
                WK_SQL_TEXT2 := 'UPDATE INDEXES (' || C_CRLF ;
            END IF;
        
            -- LOCALインデックス
            IF REC_IDX.IDX_TYPE = 'LOCAL' THEN
            
                WK_SQL_TEXT2 := WK_SQL_TEXT2 || '   ' || REC_IDX.IDX_NM || ' ' || REC_IDX.IDX_TYPE || C_CRLF ||
                                '(' || C_CRLF ||
                                '   PARTITION ' || REC_TBL.TBL_NM || '_A       TABLESPACE ' || REC_IDX.IDX_SPC_NM || C_CRLF ||
                                ',  PARTITION ' || REC_TBL.TBL_NM || '_B       TABLESPACE ' || REC_IDX.IDX_SPC_NM || C_CRLF ||
                                ',  PARTITION ' || REC_TBL.TBL_NM || '_DEFAULT TABLESPACE ' || REC_IDX.IDX_SPC_NM || C_CRLF ||
                                '),' || C_CRLF ;
            
            -- GLOBALインデックス
            ELSE
                WK_SQL_TEXT2 := WK_SQL_TEXT2 || '   ' || REC_IDX.IDX_NM || ' ' || REC_IDX.IDX_TYPE || ' TABLESPACE ' || REC_IDX.IDX_SPC_NM || ',' || C_CRLF ;
            END IF;
        
        END LOOP;
        
        -- 末尾のカンマ＋改行を')'に置換
        IF LENGTHB( WK_SQL_TEXT2 ) > 0 THEN
            WK_SQL_TEXT2 := SUBSTRB( WK_SQL_TEXT2, 1, LENGTHB( WK_SQL_TEXT2 ) - LENGTHB( ',' || C_CRLF ) ) || C_CRLF || ')' ;
        END IF;
        
        -- パーティション表への変更ALTER文作成
        WK_SQL_TEXT1 := 'ALTER TABLE ' || REC_TBL.TBL_NM || ' MODIFY' || C_CRLF ||
                        'PARTITION BY LIST( ' || C_KEY || ' )' || C_CRLF ||
                        '(' || C_CRLF ||
                        '   PARTITION ' || REC_TBL.TBL_NM || '_A       VALUES (''A'')   TABLESPACE ' || REC_TBL.TBL_SPC_NM || C_CRLF ||
                        ',  PARTITION ' || REC_TBL.TBL_NM || '_B       VALUES (''B'')   TABLESPACE ' || REC_TBL.TBL_SPC_NM || C_CRLF ||
                        ',  PARTITION ' || REC_TBL.TBL_NM || '_DEFAULT VALUES (DEFAULT) TABLESPACE ' || REC_TBL.TBL_SPC_NM || C_CRLF ||
                        ')'             || C_CRLF ||
                        WK_SQL_TEXT2    || C_CRLF ||
                        'ONLINE'        || C_CRLF ||
                        '/'             || C_CRLF ;
        
        -- SQL分出力
        DBMS_OUTPUT.PUT_LINE( WK_SQL_TEXT1 || C_CRLF || C_CRLF );
        
    END LOOP;

END;
/
```

# パーティション化確認

```SQL
-- DBAビューは、データベース内のすべてのパーティション表に関するパーティション化情報を表示します。
SELECT * FROM SYS.DBA_PART_TABLES WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME;

-- パーティション・レベルのパーティション化情報、パーティションの記憶域パラメータ
-- およびDBMS_STATパッケージまたはANALYZE文により生成されたパーティション統計
SELECT * FROM SYS.DBA_TAB_PARTITIONS WHERE TABLE_OWNER = '' ORDER BY TABLE_OWNER, TABLE_NAME, PARTITION_NAME;

-- サブパーティション・レベルのパーティション化情報、サブパーティションの記憶域パラメータ
-- およびDBMS_STATパッケージまたはANALYZE文により生成されたパーティション統計
SELECT * FROM SYS.DBA_TAB_SUBPARTITIONS WHERE TABLE_OWNER = '' ORDER BY TABLE_OWNER, TABLE_NAME, PARTITION_NAME, SUBPARTITION_NAME;

-- パーティション表のパーティション化キー列
SELECT * FROM SYS.DBA_PART_KEY_COLUMNS WHERE OWNER = '' ORDER BY OWNER, NAME, OBJECT_TYPE, COLUMN_NAME, COLUMN_POSITION;

-- コンポジット・パーティション表（およびコンポジット・パーティション表のローカル索引）のサブパーティション化キー列
SELECT * FROM SYS.DBA_SUBPART_KEY_COLUMNS WHERE OWNER = '' ORDER BY OWNER, NAME, OBJECT_TYPE, COLUMN_NAME;

-- 表のパーティションに関する列統計およびヒストグラム情報
SELECT * FROM SYS.DBA_PART_COL_STATISTICS WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME, PARTITION_NAME, COLUMN_NAME;

-- 表のサブパーティションに関する列統計およびヒストグラム情報
SELECT * FROM SYS.DBA_SUBPART_COL_STATISTICS WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME, SUBPARTITION_NAME, COLUMN_NAME;

-- 表パーティションのヒストグラムに関するヒストグラム・データ（各ヒストグラムのエンドポイント）
SELECT * FROM SYS.DBA_PART_HISTOGRAMS WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME, PARTITION_NAME, COLUMN_NAME;

-- 表サブパーティションのヒストグラムに関するヒストグラム・データ（各ヒストグラムのエンドポイント）
SELECT * FROM SYS.DBA_SUBPART_HISTOGRAMS WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME, SUBPARTITION_NAME, COLUMN_NAME;

-- パーティション索引のパーティション化情報
SELECT * FROM SYS.DBA_PART_INDEXES WHERE OWNER = '' ORDER BY OWNER, TABLE_NAME, INDEX_NAME;

-- 索引パーティションのパーティション・レベルのパーティション化情報、パーティションの記憶域パラメータ
-- DBMS_STATSパッケージまたはANALYZE文により収集された統計情報
SELECT * FROM SYS.DBA_IND_PARTITIONS WHERE INDEX_OWNER = '' ORDER BY INDEX_OWNER, INDEX_NAME, PARTITION_NAME;

-- 索引サブパーティションのパーティション・レベルのパーティション化情報、パーティションの記憶域パラメータ
-- DBMS_STATSパッケージまたはANALYZE文により収集された統計情報
SELECT * FROM SYS.DBA_IND_SUBPARTITIONS WHERE INDEX_OWNER = '' ORDER BY INDEX_OWNER, INDEX_NAME, SUBPARTITION_NAME;

-- 既存のサブパーティション・テンプレートの情報
SELECT * FROM SYS.DBA_SUBPARTITION_TEMPLATES WHERE USER_NAME = '' ORDER BY USER_NAME, TABLE_NAME, SUBPARTITION_NAME;
```
