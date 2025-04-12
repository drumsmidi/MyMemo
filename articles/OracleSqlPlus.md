# glogin設定

```SQL
--
SET TIME ON

-- SQL*Plus の 標準出力の表示時に行末の空白を出力しない
SET TRIMOUT ON

-- SQL*Plus の SPOOL 命令によるファイル書き出し時に行末の空白を出力しない
SET TRIMSPOOL ON

-- 問い合わせの結果レコード件数メッセージ、DDL の実行時の応答メッセージや PL/SQL の実行時の応答メッセージを表示する
SET FEEDBACK ON

-- 列と列の間の区切り文字を設定する
SET COLSEP '|'

-- 検索結果のヘッダを表示する
SET HEADING ON

--
SET HEADSEP '|'
SET UNDERLINE '-'

-- １ページの行数を設定する
SET PAGESIZE 50000

-- １行の長さを設定する(バイト数)
SET LINESIZE 32767

-- 数値の表示桁数を設定する
--SET NUMWIDTH 100

-- スクリプト実行によるコマンドの結果出力を表示する
SET TERMOUT ON

-- DBMS_OUTPUT による PL/SQL の標準出力を表示する
SET SERVEROUTPUT ON

-- 置換変数に設定する前後の状態を表示する
SET VERIFY ON

-- NULL を別の文字列に変換する
--SET NULL '代替文字列'

-- CSVフォーマットで出力する(12c以降)
--SET MARKUP CSV ON QUOTE ON

-- 行のラップ表示（折り返し） OFFにすると切り捨てられる模様
--SET WRAP OFF

-- ed時に開くエディターの実行パス
define _editor="D:\****\*****.exe"

-- スクリプトを実行して SELECTの結果だけを表示したり SPOOL させたい場合に使用する。
SET ECHO ON

-- 日付フォーマットの変更
ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY/MM/DD HH24:MI:SS';
```
