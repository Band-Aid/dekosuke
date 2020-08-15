---
title: Web統合
tags: 
- box 
- web統合
---

# Web統合アプリ

Box Web統合を使ってフォルダ全体をロックしてみる
Boxのフォルダの中身を全てロックしたい。こんな要望がちらほら聞こえてきます。
box Web統合APIを使って実装してみました。

実装フローはこんな感じ
![](https://cloud.box.com/shared/static/dd4e01gr2y8o4secdv4znlz00zzzg6mj.png)

一応ロック時間を指定出来るようにeFormも表示出来るようにしました。
出来上がったアプリは以下のような感じになります。
（⬇️GIFがループしないので、止まったらページをリフレッシュしてください）
![](https://cloud.box.com/shared/static/jbxsxtqdw31x09tzxsq9d785ytzldjy1.gif)


# Box Web統合ってなに？
Box上のファイルを右クリックして、色んなアクションを実行することが出来ます。
たとえば、box上のファイルをDocuSignに送ったり、共有リンクをSlack、ChatWorksなどのチャットアプリに送ったりすることが出来ます。
![](https://cloud.box.com/shared/static/gh7aoevmx9x19sw37u17pv3onrnwj8q7.png)

### Web統合を使ってみたい！
管理コンソール＞エンタープライズ設定＞アプリ
フィルタを内部アプリ選択すると、公開されているWeb統合アプリが出力されます。
お好きなものを選んでテナントに追加することができます。
![](https://cloud.box.com/shared/static/hnosq5yc5gie9ejk2lumcicnn93hvqfw.png)

# 準備
1. Box Developerコンソールにアプリケーションを作成
2. Web統合むすびつける
3. イベントハンドルするためのWebフレームワーク

# Box Developer Console
アプリの作成は[このドキュメント](https://cloud.box.com/s/4inscvj7o04jh1u0kym7ktkawcwwcrth)の1 - 7を参照

アプリの作成が終わったら統合ボタンをクリックして、Web統合を作ります。
適当な名前をつけて保存します。
![](https://cloud.box.com/shared/static/rc10sidzjybyi3h8nfip0a8q4n5ekx9v.png)

保存後、以下の設定画面が表示されているかと思います。
![](https://cloud.box.com/shared/static/0u22ggg8yc9iuwe6kqswet529gjowrue.png)

上から設定項目の説明をします。
**統合名** - 右クリックコンテキスト上に表示される名前
**説明分** - アプリの説明分
**サポートされているファイル拡張子** - サポートされている拡張子のみ右クリックコンテキストに表示するしないが選択できる（例：xlsx拡張子にのみ対応させる場合は、**サポートを特定のファイル拡張子**を選択して、xlsxと入力します
**必須の権限** - Web統合をトリガーするのに必要な権限を設定します。ここで設定した権限は、後述するAuthCodeを使って取得したAccess Tokenがを使ってできることにも影響してきます。

- [ ] <font color='red'>ダウンロード権限：</font>Web統合を実行するには、ファイルのダウンロード権限が必要。また、取得したAccess Tokenはファイルの**ダウンロード**、**プレビュー権限**のみがScopeされます。
- [ ]  <font color='red'>管理者権限：</font>管理者権限がないとWeb統合が実行出来ない。Access Tokenは上記に加えて、**ダウンロード**、**タグ**、**タスク作成**、**リネーム**、**共有**、**アップロード**のScopeが付与されます。

###統合の種類
今回はフォルダから実行したいので、フォルダーを選択します。
![](https://cloud.box.com/shared/static/2fdx4x8piektp1h5gtv4uzj7sydngrtu.png)

###コールバックの設定
Web統合をトリガー後、Boxからパラメーターを受け取るサーバを指定します。
![](https://cloud.box.com/shared/static/7zjfroqvyfaliafse9e6owwo7ijs3mgh.png)

###コールバックパラメーターの設定
本来Box APIを叩くには、<font color='red'>*1 Oauth認証</font>を済ませておく必要がありますが、今回は、Web統合トリガー時にAuth Codeをサーバに送りつけ、このAuth Codeを使って取得したAccess Tokenを使ってリソースにアクセスします。


ここで取得出来るAccess Tokenは**必須の権限**の欄で前述した、権限によって実行出来るScopeが異なります。
コールバックパラメーターとして、ドロップダウンから#auth_code#を選択し、任意のパラメーター名をつけます。
それと、今回は特定のフォルダ配下のアイテムに対してロックをかけるので、トリガー元となるFolder IDをパラメーターも指定する必要があります。
![](https://cloud.box.com/shared/static/hqyx9d50pb00h4onhrhyyb7ye2up6qhz.png)

ここまで終わったら、一度右上の保存ボタンを押下します。

#サーバーサイドの実装
Web統合を実行すると、コールバックパラメータに指定した値がboxから送られて来ます。
この送られてくるパラメータを受け取れるサーバーを実装します。

boxへのAPIリクエストは[Box SDK](https://developer.box.com/docs/box-sdks)を使うと幸せになれます。サーバの実装はそれぞれお好きなものを選択ください。
今回は、Box Ruby SDKを使います。

```gem install boxr```

まず、Boxから送られてAuthCodeからAccess Tokenを取得します。

```ruby:ruby
authCode = params['code']
folderId = params['folderid']

#受け取ったAuthorization Codeを使って、Access Tokenを取得します。
token = Boxr::get_tokens(code=authCode, grant_type: "authorization_code", assertion: nil, scope: nil, username: nil, client_id: ENV['BOX_CLIENT_ID'], client_secret: ENV['BOX_CLIENT_SECRET'])

受け取ったAccess Tokenを使ってクライアント初期化
client = Boxr::Client.new(token.access_token)
```

フォルダの中身をチェックして、ロックをかける。

```ruby:ruby
#フォルダの中身をチェック
items = client.folder_items(folderId)
        items.each {|i|
            if i.type == 'folder'
            　　#もし、TypeがFolderだったら、再度フォルダの中をチェックする
            else
           　　 #TypeがFileだったらロックをかける。client.lock_file(i.id)
            end
```

注意：Web統合経由で何らからのページを表示する際、box.comのiFrame内に表示されます。X-Frame-Optionsを許可する必要があります。

上記のような実装で、フォルダ配下のファイルに対してロックをかけるロジックの実装が可能となります。

Web統合はとても強力なツールになります。
ぜひ、いろんなソリューションを作ってみてください。


#Appendix
*1 Oauth 3 legged 認証
以下のOauth 3legged call flowを参照。
![](https://cloud.box.com/shared/static/b2ngt070963he1vns70eqgiwqxpradf6.png)