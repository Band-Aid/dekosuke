---
title: "Box CLIを使おう"
date: 2020-08-17T12:37:16+09:00
draft: false
---



# Box CLI 2.0を使おう

Box CLI 2.0の布教活動のため、このドキュメントは箱の中の人が*個人的な趣味*で書いています。
情報は随時更新される可能性がありますので、最新情報は[github](https://github.com/box/boxcli)を参照ください。

## Box CLIの歴史
2018年12月現在Box CLIには大きくわけて２つのバージョンがあります。
v1.x系統とv2.x系統があり、それぞれ大きくアーキテクチャやコマンドが異なります。
v1.xはdotnet SDKベースで開発されており、コマンドはスペース区切りでした。
一方v2.xはnodejsベースで開発されており、コマンドは`:`区切りに変わりました。
<font color="red">破壊的なイノベーション</font>が起こっています。(棒読み)

主な変更点は以下記事を参照ください。
[Box CLI 2.0がリリースされたよ](https://qiita.com/daichiiiiiii/items/f1355c779f3f6bc243d4)
[Box CLI 2.0 Changelog](https://github.com/box/boxcli/blob/master/CHANGELOG.md#200)


#**Box CLI（Command Line Interface）とは**
コマンドラインから直接一連のコマンドを使ってBoxサービスを操作し相互運用するツールです。 Box CLIを使用すると、Box APIを使用してBoxサービスとのやりとりをより迅速に行うことができ、ユーザーアカウントの作成など、ルーチンアクションまたは一括アクションを実行するためのコードを書く必要がなくなります。一連のコマンドを使用すると、コマンドラインからBox APIと対話できます


##リソース
[Github](https://github.com/box/boxcli)
[Box Developer Site](https://developer.box.com/docs/command-line-interface-cli)
[Box Developer Forum](https://community.box.com/t5/Developer-Forum/bd-p/DeveloperForum)

**それでは、Box CLI 2.0を使ってみよう！**


#Installing the Box CLI
Box CLIをインストールするには、使用しているオペレーティングシステムのバージョンを選択します。
現在インストーラーが用意されているOSの種類は
 - Windows 64bit
 - Windows 32bit
 - MacOS

インストーラーはgithubのリリースページから入手してください。
https://github.com/box/boxcli/releases

##初期設定ステップ1. Boxアプリケーションを作成する
Box Developer Consoleを使用してログインするか、無料の開発者アカウントを作成してください
[新しいアプリケーションを作成]を選択します。
「エンタープライズ統合」を選択し、「次へ」を押します。
![](https://cloud.box.com/shared/static/dfypvlb5a4axh655b1jebovbe9fxbmy3.png)

「サーバ認証」を選択し、「次へ」を押します。
*JWT認証の仕組みは[こちら](https://ja.developer.box.com/v2.0/docs/authentication-with-jwt)
![](https://cloud.box.com/shared/static/gn4kjb2528q2nr2ofel58rx65j1u8kl7.png)

アプリケーションに名前を付けます。
アプリケーション名は、Box全体で一意でなければなりません。
"アプリの作成"を押し、次に "Appを見る"を押してください。
![](https://cloud.box.com/shared/static/fuvxxlo0op8cacvrplf8ez4lv5npj7yf.png)

アプリケーションの設定を変更することで、Box CLIに与えるアクセスをカスタマイズできます。
「アプリケーションアクセス」では、「エンタープライズ」を選択すると、エンタープライズの管理作業ができます。

エンタープライズ内の既存の管理対象ユーザーとして作業する場合は、「ユーザーとしてアクションを実行する」をオンに切り替える必要があります。
「Webhooksの管理」をチェックすると、CLIはエンタープライズ内のWebhookを作成、読み取り、更新、および削除できます。
CLI内でトークン作成機能を使用するには、「ユーザーアクセストークンを生成する」をオンに切り替える必要があります。
ユーザーに成り代わって操作(as-user)を行うには、「ユーザーとして操作を実行」をオンに切り替える必要があります。
![](https://cloud.box.com/shared/static/0sov4kc1q0ei684k0j6psdh0x4ha8hpw.png)

##手順2.秘密鍵と公開鍵を生成する
デベロッパーコンソールから直接秘密鍵/公開鍵ペアを生成することができます。
**独自の暗号ペアを生成出来ますが、ここでは割愛します。詳しくは**[こちら](https://docs.box.com/docs/app-auth#section-1-generating-an-rsa-keypair)
![](https://cloud.box.com/shared/static/d4u9ckuwr3hosrq3wya75c2g2sk8j0z6.png)

##ステップ3.設定ファイルをダウンロードします。
上記ステップで公開/秘密キーペアを作成すると、設定ファイル(json)がダウンロードされます。
以下のスクリーンショットと似たようなファイルがダウンロードされてきているはずです。
![](https://cloud.box.com/shared/static/j2v0zg4qz74t76ofok3gklwaklcey3kd.png)

clientID:アプリケーションに付与される一意のID
clientSecret: アプリの認証用秘密鍵
publicKeyID: JWT認証用の一意のID
privateKey: 秘密暗号鍵
passphrase: パスフレーズ
enterpriseID: テナントID

##ステップ4.アプリケーションをテナントに結びつけをします

管理コンソールにログインし、アプリタブを開きます。
中程の新しいアプリケーションを承認ボタンを押下して下さい。
その後、ステップ3で取得したClientIDを入力してアプリケーションの承認をします
![](https://cloud.box.com/shared/static/nnanvvd854vbtj822i67ftygr4zjkhbb.png)
![](https://cloud.box.com/shared/static/6svcbbv3lziip8hpjc3b66rftkk3xce1.png)
![](https://cloud.box.com/shared/static/drdstasmjtett0ij9hg6xf84xa0wd072.png)
![](https://cloud.box.com/shared/static/0e46j0jqlrh905mg8rwywyc7lwrr5zpp.png)

##ステップ5. CLIのダウンロード、インストール、セットアップ
お使いのOS用のCLIのバージョンをダウンロードしてください。
Box CLIは、それぞれ独自のJSON構成ファイルを持つ複数の環境の設定をサポートしています。
最初の環境を設定するには、次のコマンドを実行し、JSON構成ファイルにファイルパスを指定します。
複数のBox CLI環境を追加することができます。

```
box configure:environments:add ステップ3でダウンロードした.jsonのパス --name "BoxCLI"<=好きな名前を""でくくって指定下さい
```

CLIが正しく設定されていることを確認するため、次のコマンドを使用して作業してください。
サービスアカウントのユーザー情報が表示されるはずです。
```box users:get me```


Box CLI環境一覧を表示するには、このコマンドを使用します
```box configure:environments:get```

Box CLIには、CLIがレポートファイルのダウンロードに使用するフォルダがあります。
フォルダの名前はBox Reportsになり、OSおよび現在のユーザの適切なエリアに保存されています。
Windowsの場合デフォルトだと以下フォルダにレポートは保存されます。
```C:\Users\%username%\Documents\Box-Reports```

レポートの保存場所を変えるには、

`box configure:settings --reports-folder-path=reports-folder-path`



Box CLIを使ってダウンロードしたファイルはデフォルト以下に保存されます。
```C:\Users\%username%\Downloads\Box-Downloads```

ダウンロードの保存場所を変えるには、

`box configure:settings  --downloads-folder-path=downloads-folder-path `



デフォルトだと、Box CLIの標準出力はJSONで返ってきてしまいます。
これをフォーマットされた形式で返すには以下の設定を変更します。
`box configure:settings --no-output-json`

JSONに戻すには
`box configure:settings --output-json` 



保存したレポートのデフォルトの拡張子はJSONになりますが、CSVとして保存することも出来ます。
--saveオプションに追加で--csvをつけてcsvとして保存出来ます。--jsonとつけるとJSONとしてデータを保存することも出来ます。

ユーザー一覧をCSV/JSONとして保存してみましょう。
```box users:list --save --csv/json```



#CLIコマンドとオプションの一部説明

**-h | --help**
helpオプションには、コマンドとサブコマンドの使用方法の詳細が記載されています。

**--as-user**
多くのコマンドは、Box CLI上で指定したユーザーに成り代わって実行することができます。ユーザーのIDが必要になります。ユーザーのIDは、box usersのsearchコマンドですばやく取得できます。
アプリケーションに「ユーザーとしてアクションを実行する」設定がオンになっている必要があります。

**--save** |**-s**
多くのコマンドは、Boxからレポートを作成するための保存オプションをサポートしています。これらのファイルは、デフォルトでは[Document]フォルダの[Box - Reports]に保存されます。
これは、`box configure:settings`から変更できます。

**--csv** | **--json**
saveコマンドと組み合わせて、CLIから保存されたレポートに対してjsonまたはcsvのいずれかを選択できます。

**--id-only**
このオプションを使用すると、コマンドをstdoutに基づいて連携させるのに便利です。
たとえば、Bashの場合：

```bash:bash
USER_ID="$(box users:create 橋本真也 shashimoto@test.com --id-only)" && \
FOLDER_ID="$(box folders:create 0 個人フォルダ --as-user $USER_ID --id-only)" && \
box files:upload ~/Documents/Welcome.txt --as-user $USER_ID -p $FOLDER_ID
```

**--bulk-file-path**
CLIは、いくつかのコマンドの一括処理をサポートしています。各一括処理には、一括処理を実行するBoxオブジェクトを含むファイルが必要です。

Box CLIのバルク操作に使用するCSV作成がだいぶ楽になりました。Box CLIのコマンドで使うパラメータ名がそのままCSVのヘッダーになります。作成例は[こちら](https://qiita.com/daichiiiiiii/items/f1355c779f3f6bc243d4#%E3%81%8A%E3%81%BE%E3%81%91%E3%81%9D%E3%81%AEbulk%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E4%BD%9C%E3%82%8A%E6%96%B9)

Box CLIを実行するユーザーの指定

**--as-user**
デフォルトでは、Box CLIはサービスアカウントで動作します。
そのため、特定の一般ユーザとしてファイル/フォルダの情報を操作・取得する時に
ユーザ情報を指定しないとエラーとなります。

対処としては、2つのパターンがあります。

コマンドを実行するときにその都度 "--as-user ユーザID" オプションを付けて、明示的にユーザIDを指定する。

```bash
$ box users:get me --as-user 1234567890
----Information about this user----
User Id: 1234567890
User Status: active

$ box users get:me --as-user 9876543210
----Information about this user----
User Id: 9876543210
User Status: active
```

Box CLIの環境変数にユーザIDをあらかじめ指定しておく(下記手順)。

環境変数にユーザIDをセット

```bash
$ box configure:environments:switch-user [USERID]
User set to [USERID]
#以後このユーザーで操作が開始されます
```

2.  もとのサービスアカウントに戻す場合

```
box configure:environments:switch-user --default
```



##オートコンプリート

Box CLI 2.0からCLIコマンドのタブ補完が実装されました。(**bash,zsh**シェルのみ対応)

`box autocomplete [shellの種類を選択]`

上記を実行すると、セットアップ方法が表示されます。
以下は出力結果を翻訳したものです。

1) オートコンプリート環境変数をシェルプロファイルに追加してプロファイルを更新してください
`$ printf "$(box autocomplete:script bash)" >> ~/.bashrc; source ~/.bashrc`

注意:もしあなたのシェルがログインシェルとして実行される場合は、~/.bash_profile or ~/.profileにinit scriptとして追加する必要があります。

2) さっそくやってみよう 例:

```bash
$ box <TAB><TAB>                 # コマンド補完
$ box command --<TAB><TAB>       # フラグ補完
```

##複数のEID(テナント)を接続先として登録し、切り替えて使用する

Box CLIでは、接続先テナントごとに個別の設定を持たせて、必要に応じ切り替えることができます。

例として、既存のEID-Aに加えて、新しいテナント「EID-B」を接続先としてBox CLIに追加登録する手順を記載します。

1.
EID-A側の開発者コンソールからBox CLIアプリケーションを選択
「構成」→「クライアントID」 の文字列をコピー

2.a
EID-B側のBoxの管理コンソールにアクセスし、
「アプリ」→「カスタムアプリケーション」→「新しいアプリケーションを承認」
手順1.でコピーしておいたクライアントIDを入力

3.
EID-AのBox CLIアプリケーション作成時にダウンロードしたjson ファイルをコピーして、
"enterpriseID"部分をEID-Bのものに書き換える。
jsonファイル名もわかりやすいものにリネームする(テナント名.json など)

```
test-tenant2.json
{
  "boxAppSettings": {
    "clientID": " (ApplicationのID) ",
    "clientSecret": "  (secret) ",
    "appAuth": {
      "publicKeyID": " (公開鍵のID) ",
      "privateKey": " (秘密鍵) ",
      "passphrase": "  (パスフレーズ) "
    }
  },
  "enterpriseID": "12345678"          ←← このEID部分をEID-Bのものに書き換える
}
```

4. 
Box CLI上から新しいEIDを登録

コマンド
```box configure:environments:add (Path_to_jsonfile)/xxx.json --name (Box CLI上での登録名)```

コマンド中の.jsonファイルは、手順3.で作成されたもの(EID-Bが書かれたもの)を指定
--name に渡す登録名の部分は、既存の接続先とは重複しないものを指定(企業名など、わかりやすいもの)

実行例

```
box configure:environments:add 
C:¥Users¥Admin¥BoxCLI¥test-tenant2.json --name test-tenant2
Successfully added CLI environment "test-tenant2"
```

box configure:environments:get                   # 登録が成功したか確認
Test-tenant2:
    Client ID: (app id)			  # 2つ目のテナントが追加されている 
    Enterprise ID: '12345678'	  # EIDが想定通り登録されているか確認
    Box Config File Path: C:¥Users¥Admin¥BoxCLI¥test-tenant2.json    # jsonファイル名を確認
    Has Inline Private Key: true
    Private Key Path: null
    Name: test-tenant2
    Default As-User ID: null
    Use Default As-User: false
    Cache Tokens: true



4.
現在、どの接続先設定を使っているか確認
```box configure:environments:get --current```

登録済の接続先の確認
```box configure:environments:get```

接続先の切り替え
```configure:environments:set-current (接続先として登録した名前)```

実行例

```
box configure:environments:set-current test-tenant2          #接続先の切り替えを実行
The test-tenant2 environment has been set as the default```
```

これ以降、切り替え先のEID-Bに対してBox CLIで操作可能となります。

切り替えコマンドを実行した後は、想定通りの接続先に切り替わっていることを確実に確認してください。
時間/期間を空けてから再度Box CLIでの作業に戻る際は、「box configure:environments:get -current」で現在の接続先が作業対象テナントであることを確実に確認してください。


いちど登録した接続先を削除するときは"delete"スイッチを付けて実行します。

```
box configure:environments:delete test-tenant2
The test-tenant2 environment was deleted
```

```box configure:environments:get```
(リストから削除されていることを確認)

##トークンキャッシュ

Box CLI v1.xでは、複数のEID(テナント)を切り替えて使用した場合、EIDを切り替えるごとにアクセストークンの再取得が実行されていました。
トークン再取得が実行された分だけパフォーマンスに影響が出てしまっていました。
Box CLI v2.xでは複数EIDを切り替えてもトークンのキャッシュが保持されているのでパフィーマンスの向上が見込めます。
トークンキャッシはデフォルト有効化されています。
無効化するのは、以下コマンドを実行します。

`box configure:environments:update cli_name --no-cache-tokens`

有効化するには:

`box configure:environments:update cli_name --cache-tokens`