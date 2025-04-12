# SAML認証

## 認証認可の流れ

> SP:アプリ、IDP:認証サーバー
1. ブラウザ⇒SP：Webアクセス
2. SP⇒ブラウザ：認証要求のリダイレクト指示
3. ブラウザ⇒IDP：フェデレーション認証
4. IDP⇒認証サービス：認証
5. 認証サービス⇒：アサーションの署名と発行
6. IDP⇒ブラウザ：アサーションの発行
7. ブラウザ⇒SP：アサーションの連携
8. SP⇒ブラウザ：認可後、画面表示

## Azure上での実装

App Service上でのSAML認証実装は、Azureの標準機能ではサポートされていないので
オープンソースで提供されているSAML Toolkit for JavaをWebプロジェクトに組み込んで実装する。

### SAML Toolkit for JAVA のビルド

- Java Cryptography Extension (JCE) のダウンロード

　　https://www.oracle.com/jp/java/technologies/javase-jce8-downloads.html  
　　JDK8を使用してコンパイルする場合に必要。  
　　「/[JAVA_HOME]/jre/lib/security」上の「local_policy.jar」「US_export_policy.jar」の置き換えが必要

- SAML Toolkit for Java のダウンロード

　　https://github.com/SAML-Toolkits/java-saml?tab=readme-ov-file

- Eclipseなどで、core, toolkitプロジェクトを読み込み

　　[ファイル]-[インポート]メニューより、「Maven：既存のMavenプロジェクト」を選択し上記2つのプロジェクトを開く

- 依存するJARファイルをダウンロード

　　既存のWebプロジェクトで使用しているJARライブラリとバッティングする可能性があるので  
　　作成するJAR内に依存するライブラリをリロケーションしておいた方がいい

　　https://jar-download.com/artifacts/com.onelogin/java-saml など

| JAR | 使用 | 参考 |
| --- | --- | --- |
| java-saml-x.x.x.jar | ✕ | 自前で作成 |
| java-saml-core-x.x.x.jar | ✕ | 自前で作成 |
| commons-codec-x.x.jar | 〇 | org.apache.commons.codec |
| commons-lang3-x.x.jar | 〇 | org.apache.commons.lang3 |
| jakarta.activation-api-x.x.x.jar | 〇 | javax.activation |
| jakarta.xml.bind-api-x.x.x.jar | 〇 | javax.xml.bind |
| joda-time-x.x.x.jar | 〇 | org.joda.time |
| slf4j-api-x.x.x.jar | 〇 | org.slf4j |
| stax2-api-x.x.jar | 〇 | org.codehaus.stax2 |
| woodstox-core-x.x.x.jar | 〇 | com.ctc.wstx |
| xmlsec-x.x.x.jar | 〇 | org.apache.jcp.xml.dsig.internal, org.apache.xml.security |

- java-saml, java-saml-core プロジェクトからJARファイルを発行する

　　「pom.xml」を右クリックし、[実行]-[Marven clean]、[実行]-[Marven Install]の順番で実行  
　　「target」フォルダ配下にJARファイルが出力される

　　詳細な設定は環境に合わせて設定する

java-saml - pom.xml メモ
```XML
<build>
  <plugins>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <promoteTransitiveDependencies>true</promoteTransitiveDependencies>
        <artifactSet>
          <incluedes>
          </incluedes>
        </artifactSet>
        <relocations>
          <relocation>
            <pattern>org.apache.commons.lang3</pattern>
            <shadedPattern>java-saml-core.shaded.org.apache.commons.lang3</shadedPattern>
          </relocation>
          <relocation>
            <pattern>org.apache.commons.codec</pattern>
            <shadedPattern>java-saml-core.shaded.org.apache.commons.codec</shadedPattern>
          </relocation>
          <relocation>
            <pattern>org.slf4j</pattern>
            <shadedPattern>java-saml-core.shaded.org.slf4j</shadedPattern>
          </relocation>
        </relocationgs>
      </configuration>
    </executions>
  </plugins>
</build>
```

java-saml-core - pom.xml メモ
```XML
<build>
  <plugins>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <promoteTransitiveDependencies>true</promoteTransitiveDependencies>
        <artifactSet>
          <incluedes>
            <include>org.apache.commons:commons-lang3</include>
            <include>commons-codec:commons-codec</include>
            <include>org.slf4j:slf4j-api</include>
          </incluedes>
        </artifactSet>
        <relocations>
          <relocation>
            <pattern>org.apache.commons.lang3</pattern>
            <shadedPattern>java-saml-core.shaded.org.apache.commons.lang3</shadedPattern>
          </relocation>
          <relocation>
            <pattern>org.apache.commons.codec</pattern>
            <shadedPattern>java-saml-core.shaded.org.apache.commons.codec/shadedPattern>
          </relocation>
          <relocation>
            <pattern>org.slf4j</pattern>
            <shadedPattern>java-saml-core.shaded.org.slf4j</shadedPattern>
          </relocation>
        </relocationgs>
      </configuration>
    </executions>
  </plugins>
</build>
```

- 作成したJARファイルをWebプロジェクトのライブラリへ追加する

## SP自己署名証明書を発行

- OpenSSL を使用してSP自己署名証明書を発行する

```BAT
REM 秘密鍵の作成
openssl genrsa 4096 > server.key

REM CSRの作成
openssl req -new -key server.key > server.csr

REM SP証明書の作成
openssl x509 -req -days 3650 -signkey server.key < server.csr > server.crt

REM 秘密鍵の変換（oneloginは、PKCS#8を採用しているそう。変換したsp.pemを秘密鍵として扱う）
openssl pkcs8 -topk8 -inform pem -nocrypt -in server.key -outform pem -out sp.pem
```

| 説明 | ファイル |
| --- | --- |
| 秘密鍵(PKCS#1) | server.key |
| CSR | server.csr |
| SP証明書 | server.crt |
| 秘密鍵(PKCS#8) | sp.pem |

- SSL証明書の確認

　　CSRのファイルをテキストで開き、SSL証明書の内容確認サイトなどで中身を確認する  
　　https://tech-unlimited.com/parsecrt.html

- SP証明書と秘密鍵(PKCS#8)の単一行の文字列変換

　　https://www.samltool.com/format_x509cert.php  
　　一番下の「X.509 cert in string format」の情報をコピーし、SAML構成プロパティ(onelogin.saml.properties)に設定する  

| プロパティ | 設定 |
| --- | --- |
| onelogin.saml2.sp.x509cert | server.crtの情報 | 
| onelogin.saml2.sp.privatekey | sp.pemの情報 | 


## Web実装

- Webプロジェクトへの実装は、sampleプロジェクト内にあるファイルを使用して実装する

| ファイル | プロパティ | 実装 |
| --- | --- | --- |
| dologin.jsp | IDP認証画面へのリダイレクト | 〇 | 
| dologout.jsp | SSOログアウト | ✕ | 
| acs.jsp | IDP認証後にリダイレクトするページ。ここでセッションなどにアサーション情報を設定し、SPログイン処理へリダイレクト | 〇 | 
| attrs.jsp | SAMLレスポンス情報の照会用 | ✕ | 
| sls.jsp | SP側のログアウト | ✕ | 
| metadata.jsp | IDPへ公開するメタ情報照会ページ | 〇 | 
| onelogin.saml.properties | SAML設定ファイル | 〇 | 

- onelogin.saml.properties

| プロパティ | 設定 |
| --- | --- |
| onelogin.saml2.idp.entityid | Issuer URL.IDPの情報 | 
| onelogin.saml2.idp.single_sign_on_service.url | SAML 2.0 Endpoint(HTTP).IDP認証URL | 
| onelogin.saml2.idp.single_sign_logout_service.url | SLO Endpoint(HTTP).必要に応じて設定 | 
| onelogin.saml2.idp.x509cert | X.509 Certificate | 
| onelogin.saml2.sp.entityid | Audience.metadata.jspのURLを指定 | 
| onelogin.saml2.sp.assertion_consumer_service.url | ACS (Consumer) URL Recipient.acs.jspのURLを指定 | 
| onelogin.saml2.sp.single_logout_service.url | Single Logout URL.必要に応じて設定 | 
| onelogin.saml2.sp.x509cert | X.509 Certificate.server.crtの情報 | 
| onelogin.saml2.sp.privatekey | 秘密鍵.sp.pemの情報 | 


検証用にIDPサーバーとして使用可能なサイト  
https://fujifish.github.io/samling/samling.html  

# 証明書メモ

## PFX

```BAT
REM PFXファイルから秘密鍵ファイルをエクスポート
openssl pkcs12 -in [pfxファイル] -nocerts -nodes -out [秘密鍵ファイル名] -passin pass:[パスワード]

REM PFXファイルからサーバー証明書(CRT)を抽出
openssl pkcs12 -in [pfxファイル] -clcerts -nokeys -out [crtファイル名] -passin pass:[パスワード]

REM PFXファイルから中間証明書(CA)を抽出
openssl pkcs12 -in [pfxファイル] -cacerts -nokeys -out [caファイル名] -passin pass:[パスワード]

```

# LDAP

## データ移行

LdapAdminアプリを使用して移行。Export, Inport機能で移行可能。  
但しバージョンによっては Exportファイル内で":"が"::"で出力される場合があるので注意

