VSCode
https://code.visualstudio.com/

# 環境構築手順

[参考：EclipseをやめてVisual Studio Codeに乗り換えれるか試してみる](https://qiita.com/h-r-k-matsumoto/items/406a3b48f75131a65e0a)    

## VSCode新規インストール

新規でVSCodeをインストールする場合は、「Visual Studio Code for Java」を使用する。  
[Visual Studio Code for Java](https://code.visualstudio.com/docs/languages/java#_install-visual-studio-code-for-java)

## VSCodeインストール済みの場合  

既にVSCodeをインストールする場合は、下記拡張パックをインストールする。  
- [Extension Pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)

> JDKインストールしていないと下記メッセージが表示される。  
> Get～から取得する必要はないので、環境に合わせてJDKをダウンロード＆インストールを行う。  
> <img width="337" alt="image" src="https://github.com/user-attachments/assets/ee1903d2-7ead-499a-9a36-c4832c373d74" />  

## その他、拡張パック

後は、開発環境に合わせて必要な拡張パックをインストールする。  
- [Japanese Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-ja)  
<img width="897" alt="image" src="https://github.com/user-attachments/assets/a1fb6aaf-f8da-4f43-b0b9-d79e3a070750" />

- [Spring Boot 拡張パック](https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack)  

- [Java 用 Gradle](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-gradle)  

- [コミュニティ サーバー コネクタ](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-community-server-connector)  
  Tomcatのインストールは単体で実行し、コネクタ設置で接続情報を追加する感じっぽい  

- [サーバー コネクタ](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-server-connector)  
  RHELサーバーへのCI/CD？  

- [チェックスタイル](https://marketplace.visualstudio.com/items?itemName=shengchen.vscode-checkstyle)  
  Visual Studioのヒントに相当？

- [ソナーリント](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode)  
  ソース解析ツール。Visual StudioのCode Analysisに相当？  

# GitHub連携

Gitのインストールが必要。

右クリックメニューなどから、Git GUIを開き「clone existing repository」を選択。  
Target Directoryは既存のディレクトリを指定するとクローン出来ない仕様・・・  

Gitの情報が存在するフォルダを開くとVSCodeが自動出来認識してくれる。  