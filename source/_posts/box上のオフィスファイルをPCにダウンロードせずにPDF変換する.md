---
title: box上のオフィスファイルをPCにダウンロードせずにPDF変換する
tags: 
 - box 
 - web統合
 - pdf
 - representationapi
---

ご要望の多かったBox上のファイルをWeb統合経由でPDF変換するロジックを公開します。

# はじめに
さて、Boxを使って外部にファイルを送る際、OfficeファイルをPDFに変換して共有リンクで送っているケースが多々あるかと思いますが、PDFの変換作業って時間がとてもかかる・

1. OfficeファイルをBoxからダウンロード
2. PDF変換ソフトやPDF印刷機能を使ってPDFとして保存する
3. PDFファイルをBoxにアップロードする

めんどくさいですよね(´・ω・｀)
Box上のファイルをクリックするだけで、PDF変換出来たらどれだけ楽でしょうか。

**それ、Box API使ってできます！**

![](https://cloud.box.com/shared/static/bzgzcaukmln2z4fqu66gvunixvna59k0.gif)

# Box Web統合を使ってPDF変換

以前書いたエントリーでも紹介したWeb統合を使って実装していきます。
[Box Web統合を使ってフォルダ全体をロックしてみる](https://qiita.com/daichiiiiiii/items/aee5edb6e5737a19acee)

ファイルをPDFに変換するロジックもBox APIを利用します。

実はBoxにファイルをアップロードすると、４つのファイル形式が保存されています。
1. オリジナルファイル
2. PDF変換されたファイル
3. オリジナルファイルからテキスト抽出したファイル
4. サムネイル

2〜4のファイル形式のことをBoxではRepresentation(表現、描写、表示)と呼んでいます。
Box APIを使って、PDF Representationを取得していきます。

https://developer.box.com/v2.0/reference#get-representations

このRepresentations API、呼び出し方にちょっとくせがあります( ´Д`)ﾉ
cURLコールサンプルを見ながら説明します。
まず、Representationのエンドポイントは以下の様にを呼びます。

```bash:cURL
curl -X GET \
  'https://api.box.com/2.0/files/269209725223?fields=representations' \
  -H 'Authorization: Bearer xxxxxxx' \
  -H 'x-rep-hints: [pdf]' <- HeaderのKEYにx-rep-hintsと入れて値に取得したいrepresentationのタイプを[]で囲んで呼び出します。
```

```text:一口メモ
x-rep-hintsの値は以下：
PDF:
x-rep-hints: [PDF]

32x32 JPEG Thumbnail:
x-rep-hints: [jpg?dimensions=32x32]

32x32 and 1024x1024 JPEG Thumbnails
x-rep-hints: [jpg?dimensions=32x32][jpg?dimensions=1024x1024]

32x32 JPEG and 2048x2048 PNG Thumbnail
x-rep-hints: [jpg?dimensions=32x32][png?dimensions=2048x2048]

2048x2048 JPEG or 2048x2048 PNG Thumbnail
[jpg?dimensions=2048x2048,png?dimensions=2048x2048]

Text:
x-rep-hints: [extracted_text]
```

- **返ってくるJSONレスポンス**

```json
{
    "type": "file",
    "id": "269209725223",
    "etag": "4",
    "representations": {
        "entries": [
            {
                "representation": "pdf",
                "properties": {},
                "info": {
                    "url": "https://api.box.com/2.0/internal_files/269209725223/versions/283447632979/representations/pdf"
                },
                "status": {
                    "state": "success"
                },
                "content": {
                    "url_template": "https://dl.boxcloud.com/api/2.0/internal_files/269209725223/versions/283447632979/representations/pdf/content/{+asset_path}"
                }
            }
        ]
    }
}
```

この返ってきたcontent.url_templateの値に対して更にGETリクエストを出すのですが、PDF Representationを取得する場合は末尾の{+asset_path}を除いておく必要があります。

**{+asset_path}は、サムネ取得するときに使うのですが、使用方法は改めて記事を起こします。**

```
curl -X GET \
  https://dl.boxcloud.com/api/2.0/internal_files/269209725223/versions/283447632979/representations/pdf/content/ \
  -H 'Authorization: Bearer xxxxx' \
```

上記のリクエストのレスポンスとしてPDFファイルのバイトストリームが返ってきますので、こいつをPDFファイルとして保存してあげることでPDF Representationが出来上がります。

Box Representationの使い方が分かったところで、Web統合の設定を見ていきましょう。

## Web統合の設定
![](https://cloud.box.com/shared/static/5yalz7xneu7v6id0hfyh1gr4ua84n4mz.png)

![](https://cloud.box.com/shared/static/0lxcvxfano3jlx0dhmgvw3kej5q5hoqs.png)

![](https://cloud.box.com/shared/static/qutqelm4eez98ynmpiy87e8dps3z567j.png)

![](https://cloud.box.com/shared/static/yc9fj6ygxzxaewxp8h2sg62djk7gbafx.png)

## サーバサイドの実装

今回もBox Ruby SDKを使って実装していきます。
残念ながら、Ruby SDKはまだRepresentaion APIに対応していません。
2018/1/25日現在Representation APIに対応しているSDKはJava, .net, node.jsになります。

対応しているSDKを使えばいいじゃないかって？
それもそうですが、BoxのSDKは全てOpen sourceで開発されています。
Open sourceの醍醐味。なけりゃ自分で追加する。いつかメインブランチにPULLリクエスト送ります。

追加したコードは以下：

```ruby
 def get(uri, query: nil, success_codes: [200], process_response: true, if_match: nil, box_api_header: nil, follow_redirect: true, + x_rep_hints: nil)
      uri = Addressable::URI.encode(uri)

      res = with_auto_token_refresh do
        headers = standard_headers
        headers['If-Match'] = if_match unless if_match.nil?
        headers['BoxApi'] = box_api_header unless box_api_header.nil?
      + headers['x-rep-hints'] = x_rep_hints unless x_rep_hints.nil?
        BOX_CLIENT.get(uri, query: query, header: headers, follow_redirect: follow_redirect)
      end
```

```ruby
 + def representations(file_id,representation)
 +     file_id = ensure_id(file_id)
 +     uri = "#{FILES_URI}/#{file_id}?fields=representations"
 +     file, response = get(uri, x_rep_hints: "#{representation}")
 +     file
 + end
```

では、サーバーの実装サンプルを見ていきましょう。

```ruby
require 'sinatra'
require 'boxr'
require 'rest-client'

#今回のデモでは、特にリクエスターの身元確認とかしていません。実際に実装する際は、セキュリティチェックのロジック等いれることを推奨します。
#誰からでもリクエストが飛んできしまうのを防ぐ
#実際の実装でいくつかのメソッドに分けて処理を行っていますが、ここではわかりやすさのため同じフロー内で全て処理します。

post '/convert2pdf' do
    #送られてくるFileIDパラメータを引っ張ります
    fileid = params[:fileid]

　　#AuthcodeパラメータからAccess Tokenを生成します  
    authCode = params[:code]
    atoken = Boxr::get_tokens(code=authCode, grant_type: "authorization_code", assertion: nil, scope: nil, username: nil, client_id: ENV['BOX_CLIENT_ID'], client_secret: ENV['BOX_CLIENT_SECRET']).access_token
    
begin
    #Client初期化
    client = Boxr::Client.new(atoken)

    #PDF Representationを呼び出します。
    req = client.representations(fileid,'[pdf]').representations.entries[0].content.url_template

    #取ってきたPresentation URLをダウンロードします。この際、おしりにある{+asset_path}を取り除いてあげます。
    rest = RestClient.get(req[/[^{]+/],{Authorization: "Bearer #{atoken}"})

    #PDFデータの入れ物を用意。元ファイル - 拡張子 + PDF
    boxfile = client.file_from_id(fileid).name
    filename = File.basename(boxfile,'.*')
    filePath = "./#{filename}.pdf"

    #取ってきたPDFファイルを入れ物に落とし込みます。
    File.open(filePath, 'wb'){
      |file| file.write(rest.body)
    }

    #出来上がったPDFファイルをNew versionとしてアップロード
    client.upload_new_version_of_file(filePath,fileid)

    #名前を{元の名前.pdf}に書き換えます。
    #新規バージョンアップロード時にファイル名を渡すメソッドがRuby SDKには見当たらなかったので、わざわざ書き換えています。
    #[Box API自体は、新規アップロード時に名前のプロパティ渡せます]
    #(https://developer.box.com/v2.0/reference#upload-a-new-version-of-a-file)
    client.update_file(fileid, name: "#{filename}"+".pdf")
    
    #やったよ母さん、PDFファイルがアップロード出来たよ！
    return "uploaded pdf"

    #今回作ったPDFファイルを消します。
    File.delete(filePath)
rescue Exception => e
    'エラーでたよー。困った'
    end
end
```


# 終わり
もっと良い実装方法があると思いますが、とりあえず動くサンプルとしてはご参考になるのではないかと思います。

皆さんもRepresentation APIを使ってぜひPDF変換アプリを実装してみてください！




