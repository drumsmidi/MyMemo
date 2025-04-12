# ユーザー

```SQL
-- Oracleのすべてのユーザーを確認する
SELECT * FROM ALL_USERS ORDER BY USERNAME
/

SELECT USERNAME, ACCOUNT_STATUS, LOCK_DATE FROM DBA_USERS;


-- デフォルトのプロファイルのパスワード有効期限を無期限にする。
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;

-- ユーザーパスワードの再設定
--ALTER USER TEST IDENTIFIED BY "TEST";

-- ユーザーアカウントロックの解除
--ALTER USER TEST ACCOUNT UNLOCK;
```

# オブジェクト一覧

```SQL
spo D:\Object一覧.spo

conn aaa/aaa@aaa

SELECT
    A.OWNER                                             "ユーザー"
,   A.OBJECT_TYPE                                       "オブジェクトタイプ"
,   A.OBJECT_NAME                                       "オブジェクト名"
,   SUBSTRB( RTRIM( B.COMMENTS ),1, 80 )                "コメント"
,   C.BYTES                                             "サイズ"
,   C.TABLESPACE_NAME                                   "テーブルスペース"
,   A.STATUS                                            "状態"
,   TO_CHAR( A.CREATED,'YYYY/MM/DD HH24:MI:SS' )        "作成日付"
,   TO_CHAR( A.LAST_DDL_TIME,'YYYY/MM/DD HH24:MI:SS' )  "更新日付"
FROM
    ALL_OBJECTS      A
,   DBA_TAB_COMMENTS B
,   DBA_SEGMENTS     C
WHERE
    A.OWNER                 IN ( '対象スキーマ－' )
AND A.OBJECT_TYPE           IN ( 'VIEW','PROCEDURE','FUNCTION','PACKAGE BODY' )
--AND   C.TABLESPACE_NAME   NOT IN( '' )
AND A.OWNER                 = B.OWNER(+)
AND A.OBJECT_TYPE           = B.TABLE_TYPE(+)
AND A.OBJECT_NAME           = B.TABLE_NAME(+)
AND A.OWNER                 = C.OWNER(+)
AND A.OBJECT_TYPE           = C.SEGMENT_TYPE(+)
AND A.OBJECT_NAME           = C.SEGMENT_NAME(+)
ORDER BY
    A.OWNER
,   C.TABLESPACE_NAME
,   A.OBJECT_TYPE
,   A.OBJECT_NAME
/

spo off
```

# テーブルインデックス一覧

```SQL
-- テーブル・インデックス関連付け
SELECT
    A.OWNER                     "ユーザー"
,   E.OBJECT_NAME               "テーブル名"
,   A.OBJECT_NAME               "インデックス名"
,   MAX( C.TABLESPACE_NAME )    "TBLテーブルスペース"
,   MAX( F.TABLESPACE_NAME )    "IDXテーブルスペース"
,   MAX( C.BYTES )              "サイズ"
FROM
    ALL_OBJECTS      A
,   DBA_SEGMENTS     C
,   DBA_IND_COLUMNS  D
,   ALL_OBJECTS      E
,   DBA_SEGMENTS     F
WHERE
    A.OWNER         IN ( '対象スキーマ－' )
AND A.OBJECT_TYPE   = 'INDEX'
AND A.OWNER         = C.OWNER(+)
AND A.OBJECT_TYPE   = C.SEGMENT_TYPE(+)
AND A.OBJECT_NAME   = C.SEGMENT_NAME(+)
AND A.OWNER         = D.INDEX_OWNER(+)
AND A.OBJECT_NAME   = D.INDEX_NAME(+)
AND D.TABLE_OWNER   = E.OWNER(+)
AND D.TABLE_NAME    = E.OBJECT_NAME(+)
AND E.OBJECT_TYPE   = 'TABLE'
AND E.OWNER         = F.OWNER(+)
AND E.OBJECT_TYPE   = F.SEGMENT_TYPE(+)
AND E.OBJECT_NAME   = F.SEGMENT_NAME(+)
GROUP BY
    A.OWNER
,   E.OBJECT_NAME
,   A.OBJECT_NAME
ORDER BY
    A.OWNER
,   E.OBJECT_NAME
,   A.OBJECT_NAME
/
```

# オブジェクト権限一覧

```SQL
-- ディレクトリオブジェクト権限一覧
SELECT
    * 
FROM 
    DBA_TAB_PRIVS
WHERE
    GRANTEE     IN ('') 
AND OWNER   NOT IN ('')
ORDER BY
    OWNER
,   TABLE_NAME
,   PRIVILEGE
/
```

# リコンパイル

```SQL
SELECT
    CASE OBJECT_TYPE
        WHEN 'PACKAGE BODY' THEN 'ALTER PACKAGE '               || OWNER || '.' || OBJECT_NAME || ' COMPILE BODY;'
                            ELSE 'ALTER ' || OBJECT_TYPE || ' ' || OWNER || '.' || OBJECT_NAME || ' COMPILE;'
    END
FROM
    ALL_OBJECTS
WHERE
    STATUS = 'INVALID'
ORDER BY
    OBJECT_TYPE
,   OWNER
,   OBJECT_NAME
/
```

# オブジェクト一括削除

```SQL
SET SERVEROUTPUT ON
BEGIN
    FOR REC_OBJ IN
    (
        SELECT
            A.OWNER
        ,   A.OBJECT_TYPE
        ,   A.OBJECT_NAME
        ,   B.MASTER
        FROM
            ALL_OBJECTS    A
        ,   ALL_MVIEW_LOGS B
        WHERE
            A.OBJECT_NAME = B.OBJECT_TABLE(+)
        AND A.OBJECT_TYPE IN ( 'TABLE', 'VIEW', 'PROCEDURE', 'FUNCTION', 'PACKAGE', 'TRIGGER', 'SEQUENCE', 'SYNONYM', 'MATERIALIZED VIEW' )
        AND A.OWNER IN ('')
        AND (   NOT EXISTS( SELECT 'X' FROM ALL_MVIEWS C WHERE A.OBJECT_NAME = C.MVIEW_NAME AND A.OBJECT_TYPE = 'TABLE' )
            OR  A.OBJECT_TYPE = 'MATERIALIZED VIEW' )
        ORDER BY
            A.OWNER
        ,   A.OBJECT_TYPE
        ,   NVL( B.MASTER, A.OBJECT_NAME )
        ,   B.MASTER NULLS LAST
    )
    LOOP
        IF REC_OBJ.OBJECT_TYPE = 'TABLE' THEN
            IF REC_OBJ.MASTER IS NULL THEN
                 -- 通常テーブル
                 DBMS_OUTPUT.PUT_LINE( 'DROP ' || REC_OBJ.OBJECT_TYPE || ' ' || REC_OBJ.OWNER || '.' || REC_OBJ.OBJECT_NAME || ' CASCADE CONSTRAINTS PURGE;' );
            ELSE
                 -- マテビューログ
                 DBMS_OUTPUT.PUT_LINE( 'DROP MATERIALIZED VIEW LOG ON ' || REC_OBJ.OWNER || '.' || REC_OBJ.MASTER || ';' );
            END IF
        ELSE
            -- 通常テーブル
            DBMS_OUTPUT.PUT_LINE( 'DROP ' || REC_OBJ.OBJECT_TYPE || ' ' || REC_OBJ.OWNER || '.' || REC_OBJ.OBJECT_NAME || ';' ); 
        END IF;
    END LOOP;

    -- リフレッシュグループ
    DBMS_OUTPUT.PUT_LINE( 'BEGIN' ); 

    FOR REC_OBJ IN
    (
        SELECT
            ROWNER
        ,   RNAME
        FROM
            ALL_REFRESH
        WHERE
            ROWNER IN ('')
    )
    LOOP
        DBMS_OUTPUT.PUT_LINE( 'DBMS_REFRESH.DESTROY( NAME => ''' || REC_OBJ.ROWNER || '.' || REC_OBJ.RNAME || ''');' );
    END LOOP;

    DBMS_OUTPUT.PUT_LINE( 'END;' ); 
    DBMS_OUTPUT.PUT_LINE( '/' ); 
END;
/       
```

# ディレクトリオブジェクト

```SQL
-- ディレクトリオブジェクト一覧
SELECT * FROM DBA_DIRECTORIES ORDER BY OWNER, DIRECTORY_NAME;
-- ディレクトリオブジェクト権限一覧
SELECT * FROM DBA_TAB_PRIVS WHERE TYPE = 'DIRECTORY' ORDER BY OWNER, TABLE_NAME, PRIVILEGE;
```

# トリガー

```SQL
-- トリガー一覧
SELECT OWNER, TRIGGER_NAME, STATUS FROM ALL_TRIGGERS WHERE OWNER NOT IN ('') ORDER BY OWNER, TRIGGER_NAME;

-- トリガー無効化スクリプト生成
SELECT 'ALTER TRIGGER ' || OWNER || '.' || TRIGGER_NAME || ' DISABLE;' FROM ALL_TRIGGERS
WHERE OWNER IN ('') AND STATUS = 'ENABLED'
ORDER BY TRIGGER_NAME
/

-- トリガー有効化スクリプト生成
SELECT 'ALTER TRIGGER ' || OWNER || '.' || TRIGGER_NAME || ' ENABLE;' FROM ALL_TRIGGERS
WHERE OWNER IN ('') AND STATUS = 'ENABLED'
ORDER BY TRIGGER_NAME
/
```

# シーケンス

```SQL
-- シーケンスー一覧
SELECT * FROM ALL_SEQUENCES WHERE SEQUENCE_OWNER NOT IN ('') ORDER BY SEQUENCE_OWNER, SEQUENCE_NAME;
```

# DB-LINK

```SQL
-- DB-LINK一覧
SELECT * FROM DBA_DB_LINKS ORDER BY OWNER, DB_LINK;
```
