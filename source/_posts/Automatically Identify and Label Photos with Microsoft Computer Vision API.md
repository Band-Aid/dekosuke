---
title: Automatically identiry and Lable Photos with Microsoft Computure Vision API
date: 2018-06-08 17:21:51
tags: 
- Azure 
- box 
- AI
---

# Mediumからのお引越し記事(お引越しのお引越し)
https://medium.com/p/77759913c3b9/
___

**（この記事は、box developer blogの記事Automatically Identify and Label Photos with Microsoft Computer Vision APIの日本語訳になります。）
翻訳者＝著作者になります。**


AIプラットフォームを活用することで、写真の内容に基づいて自動的に写真のインデックスを作成できます。
この例では、Box内に保存された写真をAzureのComputer Vision APIに送信し、写真のコンテキストに基づいて写真とメタデータの説明を追加する方法を説明します。

![Box内の画像をAzureに認識させる](https://cloud.box.com/shared/static/5imz6xzhuv08w0yy2erifk1csmifozah.gif)

## コールフローは以下になります。
![](https://cloud.box.com/shared/static/5hwm4aqd5y1wv0uj7h5f2nojmlzd8yum.png)

## このセットアップの前提条件：
1. Azureの開発者アカウント
2. Web統合コールバックを設定済のboxアプリケーション
3. Webコールバックを受信してリクエストを処理するWebサーバー（これは、使用する設定の種類に応じてAzureでホストすることもできます）

まず、Microsoft Azureの開発者アカウントにサインアップして、Cognitiveサービスインスタンスを起動する必要があります。 Azure Cognitive Serviceインスタンスの設定については、この[ガイド](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account)を参照してください。


Azureのサインアップしが完了たら、エンドポイントURLと0cp-Apim-Subscription-keyを書き留めておきます。 後で使用します。

ここで、Box Developerコンソールに移動し、Webインテグレーションを設定します。 クライアントのコールバックURLを指定する必要があります。 これは、ボックスWebアプリケーションからの呼び出しを受信するWebサーバーです。 ＃file_id＃をコールバックパラメータとして指定します。 この値を使用して画像をAzureに送信し、解析されたデータを使用してファイルを更新します。

![](https://cloud.box.com/shared/static/xrcnahh3tnyoy9ij1j46gpynsgo0r2d2.png)


ここからは、パラメータを受け取ってAzureに渡すアプリケーションを作成する必要があります。

Azure Computer Vision APIは解析されたデータをJSON形式で返します。 Azureの応答の例を以下に示します。

```json
{  
   "description":{  
      "tags":[  
         "sofa",
         "indoor",
         "black",
         "cat",
         "laying",
         "dog",
         "sitting",
         "white",
         "top",
         "lying",
         "blanket",
         "sleeping",
         "bed",
         "control",
         "remote"
      ],
      "captions":[  
         {  
            "text":"a cat laying on a sofa",
            "confidence":0.80696093957612369
         }
      ]
   },
   "requestId":"xxxxxxxxxx",
   "metadata":{  
      "width":3651,
      "height":2738,
      "format":"Jpeg"
   }
}

```

次の例では、Descriptionフィールドを解析し、Box Ruby SDKを使用して説明としてデータを適用します。

```ruby
require 'sinatra'
require 'boxr'
require 'rest-client'
require 'hashie'
require 'json'

post '/describe_photo' do
#Web統合リクエストからのクエリパラメータを受け取ります
fileid = params[:fileid]
        client =  Boxr::Client.new([ACCESS_TOKEN])
        dlphoto = client.download_url(fileid, version: nil)

#AzureインスタンスへのリクエストにダウンロードURLを含めます。
payload = {url: dlphoto}

＃Azureにデータを送る準備をする
request = RestClient.post 'https://westus.api.cognitive.microsoft.com/vision/v1.0/describe?maxCandidates=1', payload.to_json,headers={"content_type": "json","Ocp-Apim-Subscription-Key": "xxxxxxxxxxxxxxxxx"}

＃レスポンスを解析し、説明フィールドを抽出する
hash = JSON.parse request.body
obj = Hashie::Mash.new hash

#説明フィールドをセット
desc = obj.description.captions[0].text

#Boxのファイルに説明フィールドをアップデート
        client.update_file(fileid,description: desc.to_s)
end
```

JSONからタグを抽出して、それらのタグをメタデータとしてboxに格納することもできます。

```ruby
hash = JSON.parse request.body
           obj = Hashie::Mash.new hash
           tags = obj.description.tags

tags.each {|tag| 
metadata = {}
＃目的の値をハッシュに押し込む
＃メタデータハッシュを含め、Boxファイルをアップデートます。
client.create_metadata(fileid,metadata)
}
```

これで、Box内の写真を右クリックしてAzureに送って、画像を認識させることができます！

# APIドキュメント

上記のRubyの例の最初のAPI呼び出しは、[Box Update File Info](https://developer.box.com/reference#update-a-files-information)エンドポイントを使用しています。 上記のRubyの例の2番目のAPI呼び出しでは、Box [Metadata](https://developer.box.com/reference#create-metadata)エンドポイントに使用しています。 すべてのBox API機能を調べるには、[APIリファレンスドキュメント](https://developer.box.com/reference)を参照してください。

## Box APIを始めるにあたって

このチュートリアルでは、Box APIの可能性と機能について説明しました。 アプリケーションでBox APIをテストする場合は、ここをクリックして無料の開発者アカウントを作成してください。
このチュートリアルに関するご質問は、[Box CommunityのDeveloperフォーラム](https://community.box.com/t5/Box-Developer-Forum/bd-p/DeveloperForum)(英語)でお気軽にお問い合わせください。
