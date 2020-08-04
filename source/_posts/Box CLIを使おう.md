---
title: box cliを使おう
date: 2017-12-31 00:00:00
tags:  
 - box
 - .NETCore 
 - BoxCLI
---
Box CLIの布教活動のため、このドキュメントは以下のBox Note記載のものをQiita上に移植したものです。情報は随時更新される可能性がありますので、最新情報は下記のBox Noteを参照してください。
https://cloud.box.com/s/dy11e3nfohlc5wx817utmzba3uv7naok

もしくは、Box Developer Siteを参照ください
https://developer.box.com/docs/command-line-interface-cli


# ボックスコマンドラインインターフェイス

**Box CLI（Command Line Interface）とは**
コマンドラインから直接一連のコマンドを使ってBoxサービスを操作し相互運用するツールです。 Box CLIを使用すると、Box APIを使用してBoxサービスとのやりとりをより迅速に行うことができ、ユーザーアカウントの作成など、ルーチンアクションまたは一括アクションを実行するためのコードを書く必要がなくなります。一連のコマンドを使用すると、コマンドラインからBox APIと対話できます。

## Installing the Box CLI
Box CLIをインストールするには、使用しているオペレーティングシステムのバージョンを選択します。

[Windows](https://cloud.box.com/s/iqhx409vo5pngncwlhxn8icoo15lsz6y)
[Mac 10.12以上](https://cloud.box.com/s/grpn7d45yi10gtauadzdcfyuf3k0cf00)
[Mac 10.11以下](https://cloud.box.com/s/cjlnewdmxlviuo9e1vsjrenrs1oqf5u3)

"**Mac 10.11またはそれ以降を使用している場合は、インストールする前にOpenSSLのバージョンが最新であることを確認する必要があります。**"

OpenSSLのバージョンをアップデートするには、以下の手順に従ってください：

インストールされていない場合はHomebrewをインストールしてください

```bash
/usr/bin/ruby -e "$（curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install）"
```

ターミナルから次のコマンドを実行します。


brewによる更新

```bash
brew install openssl
mkdir -p / usr / local / lib
ln -s /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib / usr / local / lib /
ln -s /usr/local/opt/openssl/lib/libssl.1.0.0.dylib / usr / local / lib /
```


<details><summary>**Linuxでのビルド手順は以下になります。**</summary><div>
1. .net core SDKをMSFTサイトからダウンロード、インストール下さい\
　・各種ディストリに合わせてインストール手順を以下サイトよりご参照下さい
　・https://www.microsoft.com/net/core#linuxubuntu
2. https://github.com/box/boxcliのソースをローカルにクローン
3. BoxCLI/BoxCLIフォルダに移動        下記コマンドを実行してビルド下さい
　・dotnet build -r <target-runtime>
　・<target-runtime>を指定のOS versionに書き換えて下さい。例：ubuntu.16.04-x64, osx.10.12-x64
RID一覧：https://docs.microsoft.com/ja-jp/dotnet/core/rid-catalog
・バイナリは./bin/Debug/netcoreapp2.0/<target-runtime>/boxに配置されます。
・本番環境用にビルドする場合は、以下コマンドを実行下さい。
・dotnet publish --configuration Release --runtime <target-runtime>
・バイナリ./bin/Release/netcoreapp2.0/<target-runtime>/publishに配置されます。
4. 上記で作成したバイナリへパスを通して下さい。</div></details>

# リソース
[Github](https://github.com/box/boxcli)
[Box Developer Site](https://developer.box.com/docs/command-line-interface-cli)
[Box Developer Forum](https://community.box.com/t5/Developer-Forum/bd-p/DeveloperForum)

# 初期設定ステップ1. Boxアプリケーションを作成する
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

# 手順2.秘密鍵と公開鍵を生成する
デベロッパーコンソールから直接秘密鍵/公開鍵ペアを生成することができます。
**独自の暗号ペアを生成出来ますが、ここでは割愛します。詳しくは**[こちら](https://docs.box.com/docs/app-auth#section-1-generating-an-rsa-keypair)
![](https://cloud.box.com/shared/static/d4u9ckuwr3hosrq3wya75c2g2sk8j0z6.png)

# ステップ3.設定ファイルをダウンロードします。
上記ステップで公開/秘密キーペアを作成すると、設定ファイル(json)がダウンロードされます。
以下のスクリーンショットと似たようなファイルがダウンロードされてきているはずです。
![](https://cloud.box.com/shared/static/j2v0zg4qz74t76ofok3gklwaklcey3kd.png)

clientID:アプリケーションに付与される一意のID
clientSecret: アプリの認証用秘密鍵
publicKeyID: JWT認証用の一意のID
privateKey: 秘密暗号鍵
passphrase: パスフレーズ
enterpriseID: テナントID

# ステップ4.アプリケーションをテナントに結びつけをします

管理コンソールにログインし、アプリタブを開きます。
中程の新しいアプリケーションを承認ボタンを押下して下さい。
その後、ステップ3で取得したClientIDを入力してアプリケーションの承認をします
![](https://cloud.box.com/shared/static/nnanvvd854vbtj822i67ftygr4zjkhbb.png)
![](https://cloud.box.com/shared/static/6svcbbv3lziip8hpjc3b66rftkk3xce1.png)
![](https://cloud.box.com/shared/static/drdstasmjtett0ij9hg6xf84xa0wd072.png)
![](https://cloud.box.com/shared/static/0e46j0jqlrh905mg8rwywyc7lwrr5zpp.png)

# ステップ5. CLIのダウンロード、インストール、セットアップ
お使いのOS用のCLIのバージョンをダウンロードしてください。
Box CLIは、それぞれ独自のJSON構成ファイルを持つ複数の環境の設定をサポートしています。
最初の環境を設定するには、次のコマンドを実行し、JSON構成ファイルにファイルパスを指定します。


`box configure environments add` ステップ3でダウンロードした.jsonのパス --name BoxCLI<=好きな名前を指定下さい

CLIが正しく設定されていることを確認するため、次のコマンドを使用して作業してください。
サービスアカウントのユーザー情報が表示されるはずです。
`box users get me`


現在のCLI環境設定を表示するには、このコマンドを使用します
`box configure environments get-current`


Box CLIには、CLIがレポートファイルのダウンロードに使用するフォルダがあります。
ダウンロードフォルダの名前はBox Reportsになり、OSおよび現在のユーザの適切なダウンロードエリアに保存されています。
Windowsの場合デフォルトだと以下フォルダにレポートは保存されます。
`C:\Users\%username%\Documents\Box-Reports`

保存したレポートのデフォルトの拡張子はJSONになりますが、CSVとして保存することも出来ます。
--saveオプションに追加で--file-formatをつけてcsvとして保存出来ます。

ユーザー一覧をCSV*として保存してみましょう。
**BoxCLI v1.0.2未満では出力されたCSVファイルにUTF-8ヘッダーが付与されていないため、Excelで開くと日本語が文字化けを起こします。Excelで正しく開くには、テキストエディタ等を使用してUTF-8ヘッダーを付与もしくは、Box CLIをv1.0.2以上にアップグレードしてご利用ください。**
`box users list --save --file-format csv`


# CLIコマンドとオプションの一部説明
-h | --help
helpオプションには、コマンドとサブコマンドの使用方法の詳細が記載されています。

--as-user
多くのコマンドは、Box CLI上で指定したユーザーに成り代わって実行することができます。ユーザーのIDが必要になります。ユーザーのIDは、box usersのsearchコマンドですばやく取得できます。
アプリケーションに「ユーザーとしてアクションを実行する」設定がオンになっている必要があります。

--save
多くのコマンドは、Boxからレポートを作成するための保存オプションをサポートしています。これらのファイルは、デフォルトでは[Document]フォルダの[Box - Reports]に保存されます。
これは、box environments configureから変更できます。
--file-format
saveコマンドと組み合わせて、CLIから保存されたレポートに対してjsonまたはcsvのいずれかを選択できます。

--json
このオプションを使用すると、APIの応答がJSON形式でstdoutに出力されます。
box folders list-items 0 --json > 〜/Documents/folders-list-items-0.json

--id-only
このオプションを使用すると、コマンドをstdoutに基づいて連携させるのに便利です。
たとえば、Bashの場合：

```bash
USER_ID="$(box users create 橋本真也 shashimoto@test.com --id-only)" && \
FOLDER_ID="$(box folders create 0 個人フォルダ --as-user $USER_ID --id-only)" && \
box files upload ~/Documents/Welcome.pptx --as-user $USER_ID -p $FOLDER_ID

```

または、PowerShellの場合：

```powershell
$FILE_PATH = "C:\Users\Daichi ishida\Documents\Welcome.pptx"
$USER_ID = box users create 橋本真也 shashimoto@test.com --id-only;
$FOLDER_ID = box folders create 0 個人フォルダ --as-user $USER_ID --id-only;
box files upload $FILE_PATH --as-user $USER_ID -p $FOLDER_ID;
```

--bulk-file-path
CLIは、いくつかのコマンドの一括処理をサポートしています。各一括処理には、一括処理を実行するBoxオブジェクトを含むファイルが必要です。

注意：一括削除アクションはプロンプトを無視し、Boxオブジェクトがこれらの機能をサポートしている場合は、通知しないようにし、強制的に削除します。

例えば：
`box users create --bulk-file-path ~/Documents/Box-Reports/create-users.json`
CLIはファイル形式がJSONであることを自動的に検出します。 JSONは次の書式に従わなければなりません：

```json
{
    "entries"：[
        ...
        {
            "name"： "橋本真也",
            "login"："shashimoto@test.com"
        },
        {
            "name"： "岸辺",
            "login"："kshiro@test.com"
        }
        ...
    ]
}
```

バルクアクションをサポートするdeleteコマンドとupdateコマンドの場合、Boxオブジェクトは通常、オブジェクトのIDを含める必要があります。削除の場合、通常はIDが必要です。

`box users delete --bulk-file-path 〜/Documents/delete-users.json`

```
{
    "entries"：[
        ...
        {
            "id"： "2362696581"
        },
        {
            "id"： "2362696772"
        }
...
```

注：CSVファイルは一括処理でもサポートされていますが、特別なCSVテンプレートが必要です。


Box CLIを実行するユーザーの指定

デフォルトでは、Box CLIはサービスアカウントで動作します。
そのため、特定の一般ユーザとしてファイル/フォルダの情報を操作・取得する時に
ユーザ情報を指定しないとエラーとなります。

対処としては、2つのパターンがあります。

コマンドを実行するときにその都度 "--as-user ユーザID" オプションを付けて、明示的にユーザIDを指定する。

```bash
$ box users get me --as-user 1234567890
----Information about this user----
User Id: 1234567890
User Status: active

$ box users get me --as-user 9876543210
----Information about this user----
User Id: 9876543210
User Status: active
```

Box CLIの環境変数にユーザIDをあらかじめ指定しておく(下記手順)。

環境変数にユーザIDをセット

```bash
$ box configure environments set-default-as-user ユーザID
Successfully set the Default User ID
```

2.  当該のユーザIDでセッションを開始

```bash
$ box sessions start-user-session
Session started for User XXXXXXXXX. Session expires at 2017/12/12 2:40:04
※セッションの有効期限は1時間
以降、--as-userオプションの指定不要でコマンドを実行可能となります。
```

3. セッションの終了
```
$ box sessions end-user-session
All sessions ended.
```


複数のEID(テナント)を接続先として登録し、切り替えて使用する

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

```json
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
`box configure environments add (Path_to_jsonfile)/xxx.json --name``` (Box CLI上での登録名)

コマンド中の.jsonファイルは、手順3.で作成されたもの(EID-Bが書かれたもの)を指定
--name に渡す登録名の部分は、既存の接続先とは重複しないものを指定(企業名など、わかりやすいもの)

実行例

```bash
box configure environments add C:¥Users¥Admin¥BoxCLI¥test-tenant2.json --name test-tenant2
<省略>
Successfully configured new Box environment.

box configure environments list                    # 登録が成功したか確認
*******************************
Name: boxcli
Client ID: (AppのID)
Enterprise ID: (最初に登録したテナントのEID)
Path to Config File: (読み込ませたjsonファイルのパス)
Path to Private Key File:
Has in-line Private Key?: Yes
*******************************
*******************************
Name: test-tenant2                                                # 2つ目のテナントが追加されている 
Client ID: (AppのID)
Enterprise ID:  12345678                                   #EIDが想定通り登録されているか確認
Path to Config File: C:¥Users¥Admin¥BoxCLI¥test-tenant2.json    # jsonファイル名を確認
Path to Private Key File:
Has in-line Private Key?: Yes
*******************************
```

4.
現在、どの接続先設定を使っているか確認
`box configure environments get-current`

登録済の接続先の確認
`box configure environments list`

接続先の切り替え
`box configure environments set-current (接続先として登録した名前)`

実行例

```bash
box configure environments set-current test-tenant2          #接続先の切り替えを実行
Successfully set new default environment:
Current default environment:
Name:  test-tenant2                            # 切り替え後の接続設定名を確認
Client ID: (AppのID)
Enterprise ID: 12345678                    # 切り替え後のEIDを確認
```

これ以降、切り替え先のEID-Bに対してBox CLIで操作可能となります。

切り替えコマンドを実行した後は、想定通りの接続先に切り替わっていることを確実に確認してください。
時間/期間を空けてから再度Box CLIでの作業に戻る際は、「box configure environments get-current」で現在の接続先が作業対象テナントであることを確実に確認してください。


いちど登録した接続先を削除するときは"delete"スイッチを付けて実行します。

```bash
box configure environments delete test-tenant2
Are you sure you want to delete this environment? y/N  (yを応答)
Successfully deleted environment
```

`box configure environments list`
(リストから削除されていることを確認)


