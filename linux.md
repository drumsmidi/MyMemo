
[Red Hat Enterprise Linux](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9)

# 基礎的なこと

## CRON

```SHELL
# cron設定編集
crontab -u [設定名] -e

# cron設定確認
crontab -u [設定名] -l
```

## bash_profile

各システムユーザーの「.bash_profile」は要確認  
環境変数の設定など行っている。  

## umask

各システムユーザーのデフォルトパーミッションの設定確認

```SHELL
# 現在の設定確認
umask

# UMASK値更新
umask 004
```

ディレクトリ、ファイル新規作成時のデフォルトの権限から
UMASKの値を引いた権限でディレクトリ、ファイルが作成される


## systemmd

システムの自動起動・停止設定

[systemd の管理](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-systemd_configuring-basic-system-settings#managing-systemd_configuring-basic-system-settings)

/etc/systemd/system/[サービス名].service
```Shell
[Unit]
Description=[サービスの説明]
After=[別サービス名].service   # 指定したサービス起動後に自身のサービスを起動する
Before=[別サービス名].service  # 自身のサービス起動後に起動するサービス

[Service]
# ユニットの起動タイミング
# simple  : プロセスが起動した時点で起動完了
# forking : フォークして親プロセスが終了した時点で起動完了とする
# oneshot : 次のユニットを実行する前に自身のプロセスを終了する
# dbus    : D-Bus を使うプロセスで、D-Bus の接続名を見つけると起動完了
# notify  : d_notify() 関数で起動完了のメッセージを受け取ったときに起動完了とする
# idle    : 他のジョブが終了するまで待機する
Type=simple

ExecStart=[起動時に実行するシェルパス等]
ExecStop=[停止時に実行するシェルパス等]

# always     : 常に再起動
# no         : 再起動しない
# on-success : 終了コードが0の際に再起動する
# on-failure : 終了コードが0以外の際に再起動する
Restart=always

[Install]
# systemctl enable時の動作
WantedBy=multi-user.target
```

```Shell
# システムサービスのリスト表示
# 現在ロードされているすべてのサービスユニットをリストし、使用可能なすべてのサービスユニットのステータスを表示できます。
systemctl list-units --type service

# システムサービスステータスの表示
systemctl status [サービス名].service

# systemd ユニットの起動と停止
systemctl start [サービス名].service

# systemd システムサービスの停止
systemctl stop [サービス名].service


```

# Tomcat

## 起動設定

```Shell
# メモリの70%を設定
MEM_TOTAL_KB='cat /proc/meminfo | grep MemTotal | awk '{print $2}'
MEM_TOTAL='expr ${MEM_TOTAL_KB} / 1024'
HEEP_SIZE='expr ${MEM_TOTAL} \* 70 / 100'
CATALINA_OPTS=${CATALINA_OPTS} -Xms${HEEP_SIZE}m -Xmx${HEEP_SIZE}m -xx:MetaspaceSize=256m -xx:MaxMetaspaceSize=512m
```

メモ  
- Tomcatのバージョンを上げた際に、UMASK の設定が変わりログファイルの権限が変わることがあった。  
確か、Tomcatの起動シェル内で UMASKの環境変数が未設定の場合に設定される初期値が変わっていた。  


## ログローテーション

```SHELL
# /etc/logrotate.d/tomcat
/opt/tomcat/logs/catalina.out
{
    daily                      # 1日1回ローテーションを実行
    notifempty                 # ログファイルが空の場合、ローテーションしない
    rotate 10                  # 世代数
    missingok                  # 指定のログファイルが存在していなくてもエラーを出さない
    create 0644 [user] [group] # ログファイルの権限
}

# ローテーション即時実行
logrotate -f /etc/logrotate.d/tomcat

# ローテーションが実行されるタイミング確認
ls /etc/cron*
cat /etc/anacrontab
```


# ファイル移行

## 別サーバーへのファイル移行

```SHELL
# 圧縮
find -L /path -type f -not \( -path '*.xml' \) -print0 | tar czvfp data.tar.gzip --null -T -

# 解凍
cd /
tar xfvzk data.tar.gzip
```

> 空のディレクトリは移行されないので、ディレクトリの補完が必要

### tar オプション

| オプション | 説明 |
| --- | --- |
| c | アーカイブを作成 |
| z | gzip形式 |
| v | 処理中のファイル名を表示 |
| f | アーカイブファイル名指定 |
| p | パーミッションを保持 |
| x | アーカイブを展開 |
| k | 展開時に既存のファイルを上書きしない |


## ディレクトリ移行

```SHELL
# ディレクトリ一覧
find -L /path -type d | xargs ls -la

mkdir [path]
chown [user]:[group] [path]
chmod 777 [path]
```

## ショートカット移行

```SHELL
# ディレクトリ一覧
find -L /path -type l | xargs ls -la

ln -s [ショートカット先パス] [ショートカットパス]
```

## SSH/SCP

```SHELL
# うろ覚え、リモートからローカルにコピーする際には、先にローカルにディレクトリを作成しておく
SSH [接続ユーザー]@[ホスト] find -L [パス] -type d | xargs -IXXX mkdir /home/test/XXX

# リモート先の指定は、[接続ユーザー]@[ホスト]:[パス]
SCP -R [コピー元] [コピー先]
```


# ファイル圧縮

```SHELL
# 前日、前月の日付取得
BEFORE_DATE_YYYYMMDD=${date +'%Y%m%d' --date '1 day ago')
BEFORE_DATE_YYYY_MM_DD=${date +'%Y-%m-%d' --date '1 day ago')
BEFORE_DATE_YYYYMM=${date -d "$(date +'%Y%m01') 1 month ago" +'%Y%m')
BEFORE_DATE_YYYY_MM=${date -d "$(date +'%Y%m01') 1 month ago" +'%Y-%m')

L_LOG_DIRPATH=/home/log
L_LOG_SEARCHTEXT=*${BEFORE_DATE_YYYY_MM}*
L_ARCHIVE_DIRPATH=/home/log/bk
L_ARCHIVE_FILENAME=aaa_$BEFORE_DATE_YYYYMM.tar.gz

cd $L_LOG_DIRPATH
pwd

if [ -f "${L_ARCHIVE_DIRPATH}/${L_ARCHIVE_FILENAME}" ]; then
  echo "圧縮ファイルが既に存在します"
  exit 0
fi

# 圧縮
if ! find . -maxdepth 1 -type f -name "${L_LOG_SEARCHTEXT}" -print0 | tar czvf ${L_ARCHIVE_DIRPATH}/${L_ARCHIVE_FILENAME} -C $L_LOG_DIRPATH --null -T -; then
  echo "圧縮ファイル作成失敗"
  exit 1
fi

# 圧縮元ファイル削除
for file in $(find . -maxdepth 1 -type f -name "${L_LOG_SEARCHTEXT}" -print0); do
  echo "ファイル削除=${file}"
  if ! rm -f $file; then
    echo "ファイル削除失敗"
    exit 2
  fi
done

# 圧縮ファイル権限変更
if ! chmod 766 $L_ARCHIVE_DIRPATH/$L_ARCHIVE_FILENAME; then
  echo "圧縮ファイル権限変更失敗"
  exit 3
fi
```