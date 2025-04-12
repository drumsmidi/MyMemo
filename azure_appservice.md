# Azure CLIコマンド

```
REM ログイン
az login
REM 環境変数の設定
az webapp config appsettings set --resource-group [リソースグループ] --name [リソース] --settings [変数名]=[値]
REM デプロイ
az webapp deploy --resource-group [リソースグループ] --name [リソース] --src-path ./[package-name].war --type war --async true --clean
REM スワップ
az webapp deployment slot swap --resource-group [リソースグループ] --name [リソース] --slot [運用：prod、その他：スロット名]
```

# WARファイルのデプロイ

- AzureCLIからデプロイした場合、/api/publish APIでのデプロイ処理となる  
　デプロイを実施すると「/home/site/wwwroot/app.war」にWARファイルが格納される。  
　その後、「/usr/local/tomcat/webapps/ROOT.war」へコピーされ展開される。  

- /usr配下の編集はできないので、tomcatのconf関連の設定を自前で修正する場合は  
　/home/tomcat/conf フォルダなどに設定ファイルをコピーして、CATALINA_BASE＝/home/tomcatを指定する。  
　但し、上記設定を使用する場合は、Apache Tomcatの自動更新は無効にしておく必要がある。  

# スワップ

スワップ機能を使用すると /home配下の全てがスロット間で切り替わってしまうので、使用しない方がよさそう。  
使用する場合は、外部ストレージにログ出力するなど対応が必要。  

# 環境変数

[Azure App Service の環境変数とアプリ設定](https://learn.microsoft.com/ja-jp/azure/app-service/reference-app-settings?tabs=kudu%2Cdotnet)

```SHELL
-- タイムゾーンを変更
WEBSITE_TIME_ZONE = Asia/Tokyo

-- スワップ切り替え時に再起動する場合がある。この再起動を最小限に抑えるための設定（らしい）
WEBSITE_ADD_SITENAME_BINDINGS_IN_APPHOST_CONFIG = 1

-- 継続的な負荷上昇を検知した際に、回復動作が実行されるのを回避しておく
WEBSITE_PROACTIVE_AUTOHEAL_ENABLED = false

-- TOMCAT関連
CATALINA_OPTS =
CATALINA_BASE = /home/tomcat
```

# ロケール

App Serviceは、初期設定が「en-US」なので「ja-JP」設定に変更する。  
優先順：LANGUAGE > LC_ALL > LC_xxx > LANG

```
LANG = ja_JP.UTF-8
LANGUAGE = 
LC_ADDRESS = ja_JP.UTF-8
LC_ALL = 
LC_COLLATE = ja_JP.UTF-8
LC_CTYPE = ja_JP.UTF-8
LC_IDENTIFICATION = ja_JP.UTF-8
LC_MEASUREMENT = ja_JP.UTF-8
LC_MESSAGES = ja_JP.UTF-8
LC_MONETARY = ja_JP.UTF-8
LC_NAME = ja_JP.UTF-8
LC_PAPER = ja_JP.UTF-8
LC_TELEPHONE = ja_JP.UTF-8
LC_TIME = ja_JP.UTF-8
```

# File Manager

[開発ツール]-[高度なツール]より、[移動→]をクリック  
URLの末尾に「/newui」を指定してアクセスすると File Managerが使用できる。  

調査などでのログファイル参照する際などに使用する。

サイズが大きいファイルはアップロードできないので、[デプロイメント]-[デプロイセンター]メニューの「FTPS 資格情報」を使用した方がいい

# AppService疎通確認

## App Service on Linux

```Shell
apk update
apk add tcptraceroute
cd /usr/bin
wget http://www.vdberg.org/~richard/tcpping
chmod 755 tcpping
apk add bc

tcpping [IP] [Port]
```

# Azure環境の別リソースへのアクセス

VNET(Virtual Network)統合を行う必要がある。

[リモート リソースへの接続性](https://learn.microsoft.com/ja-jp/azure/app-service/overview-security#connectivity-to-remote-resources)


# その他メモ

システムが停止しているに変わらず、メトリックなどを見た場合に毎日特定の時間帯に負荷が上昇していることがある。

これは、AppServiceのOS側の処理で、日々負荷が上るタイミングがある


