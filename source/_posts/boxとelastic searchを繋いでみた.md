---
title: boxとelastic searchをつなぎでみた
date: 2018-06-08 10:00:00
tags: 
- box 
- elasticsearch
---
# boxとelasticsearch

boxの検索には、制限がある。
box community記事より抜粋:

`Box は、ファイル内のキーワードに依存して、文書ごとに 1 万の全文検索文字を格納します。Box の検索索引では、多数の異なるトークン化およびストップワード削除フィルタが使用されます。`

つまり、１万バイトを超える文書内の文字列については検索対象とならず、boxの検索候補に出てきません。
毎日使っている自分からすると、実はそんなにこの制限のせいで困ったことはないです。
大抵探している単語ってのは、文章中の1万バイト未満で出てくるので、多くのドキュメントは探せます。

ファイルサーバーを使ってた時代は、検索すらできなかったので(できても数時間放置してやっと検索結果が全部出揃うレベル)、それに比べたら全然マシ。

しかし、本当にフルテキスト検索ができないと困るという人もいる。
よく聞くのが、保険外交員や研究員が保険適用の過去事例や研究論文を探し出すのにフルテキスト検索ができないと困るそうです。

いちおうBoxでも検索可能ファイルサイズを大きくするという噂はあるけどいつになるかわからない。
そしたら、検索システムと連携して自分で作っちまおうということでelastic searchの登場です。
boxとelastic searchを連携してFull Text検索を実現しちゃうおう。
以下アプリは他のbox社員がPoCで作成したアプリケーションになります。
[box elastic search](https://github.com/kylefernandadams/box-elasticsearch)

注意：これはコンセプトアプリケーションです。本番環境での運用には気をつけてください。理由は後述します。


##  事前準備
以下を事前に入れておく必要があります。
elasticsearch
kibana
node
yarn

## 導入方法

### 1. 以下のレポジトリをclone
`git clone https://github.com/kylefernandadams/box-elasticsearch`

### 2. 必要node packageをインストール

```bash
13:26:41 ~/git/box-elasticsearch 
$ npm install

> fsevents@1.2.3 install /Users/xxx/git/box-elasticsearch/node_modules/fsevents
> node install

[fsevents] Success: "/Users/xxx/git/box-elasticsearch/node_modules/fsevents/lib/binding/Release/node-v59-darwin-x64/fse.node" is installed via remote

> nodemon@1.17.3 postinstall /Users/xxx/git/box-elasticsearch/node_modules/nodemon
> node -e "console.log('\u001b[32mLove nodemon? You can now support the project via the open collective:\u001b[22m\u001b[39m\n > \u001b[96m\u001b[1mhttps://opencollective.com/nodemon/donate\u001b[0m\n')" || exit 0

Love nodemon? You can now support the project via the open collective:
 > https://opencollective.com/nodemon/donate

npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN box-elasticsearch@1.0.0 No repository field.
added 555 packages in 13.565s
```

### 3. boxのjwt jsonを取得する

Box CLIを使おうの[このあたり](https://qiita.com/daichiiiiiii/items/93d4155ff93561bcac8f#%E5%88%9D%E6%9C%9F%E8%A8%AD%E5%AE%9A%E3%82%B9%E3%83%86%E3%83%83%E3%83%971-box%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)を参照して[JSONファイル](https://qiita.com/daichiiiiiii/items/93d4155ff93561bcac8f#%E5%88%9D%E6%9C%9F%E8%A8%AD%E5%AE%9A%E3%82%B9%E3%83%86%E3%83%83%E3%83%971-box%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)をダウンロードしてください。

**↑スコープに注意：**
エンタープライズスコープを許可してください。



ダウンロードしたら、box_config.jsonとリネームします。
![](https://cloud.box.com/shared/static/w1cgk6s2599ca4obgi5suactw5u8kcag.png)

リネームしたら、box-elasticsearchフォルダのルートに置きます。
![](https://cloud.box.com/shared/static/xiwe0kcf6xiydkfitbeh8nm6opr0fmec.png)

### 4. 検索設定

このデモアプリでは、boxエンタープライズイベントから検索対象のアイテムを取得してきます。
つまり、テナント内のすべてのドキュメントが検索対象となってしまう恐れがあります。ElasticSearchにアクセス権があるユーザーはすべてのドキュメントが見れてしまいます。
検索対象とする日付やファイルタイプは以下設定ファイルで調整できます

app_config.json

```json
{
    "box": { 
        "serviceAccount": "dishida+newmail@box.com",
        "pollingInterval": 60,
        "lookbackDuration": "month",
        "lookbackValue": 1,
        "textFileExtensions": [
            ".as", ".as3", ".asm", ".bat", ".boxnote", ".c", ".cc", ".cmake", ".cpp", ".cs", ".css", 
            ".csv", ".cxx", ".diff", ".doc", ".docx", ".erb", ".gdoc", ".groovy", ".gsheet", ".h", ".haml", 
            ".hh", ".htm", ".html", ".java", ".js", ".less", ".m", ".make", ".ml", ".mm", ".msg", ".odp", ".ods", 
            ".odt", ".pdf", ".php", ".pl", ".ppt", ".pptx", ".properties", ".py", ".rb", ".rtf", ".sass", ".scala", ".scm", 
            ".script", ".sh", ".sml", ".sql", ".txt", ".vi", ".vim", ".wpd", ".xls", ".xlsm", ".xlsx", ".xml", 
            ".xsd", ".xsl", ".yaml", ".bmp", ".gif", ".jpeg", ".jpg", ".png", ".psd", ".svg", ".tif", 
            ".tiff", ".svs", ".tga"
        ]
    },
    "elastic": {
        "host": "localhost:9200",
        "logLevel": "debug",
        "indexName": "box",
        "filesType": "files"
    }
}
```

### 5. indexスタート！

まず、elastic searchとkibanaのサービスをスタートします。
使用しているOSによってサービス起動方法は異なるので、各自調べてくださいな
[elasticsearch](http://www.elastic.co/guide/en/elasticsearch/guide/current/running-elasticsearch.html)
[kibana](https://www.elastic.co/guide/en/kibana/current/setup.html)

上記２つのサービスが無事起動したら、今回のアプリを起動します

`yarn start `
起動すると、boxのエンタープライズイベントを検知して以下のようにガンガンindexがかかるはずです。

```bash
13:56:45 ~/git/box-elasticsearch 
$ yarn start
yarn run v1.6.0
(node:9869) [DEP0005] DeprecationWarning: Buffer() is deprecated due to security and usability issues. Please use the Buffer.alloc(), Buffer.allocUnsafe(), or Buffer.from() methods instead.
$ npm run build && node dist/index.js

> box-elasticsearch@1.0.0 build /Users/xxx/git/box-elasticsearch
> rimraf dist/ && babel ./ --out-dir dist/ --ignore ./node_modules,./.babelrc,./package.json,./npm-debug.log --copy-files

events/events-parser.js -> dist/events/events-parser.js
events/events-poller.js -> dist/events/events-poller.js
events/text-extractor.js -> dist/events/text-extractor.js
index/events-indexer.js -> dist/index/events-indexer.js
index/index-manager.js -> dist/index/index-manager.js
index.js -> dist/index.js
Elasticsearch INFO: 2018-05-01T05:11:15Z
  Adding connection to http://localhost:9200/

Elasticsearch INFO: 2018-05-01T05:11:15Z
  Adding connection to http://localhost:9200/

Starting box elasticsearch example...
Checking if ES index exists...
Elasticsearch DEBUG: 2018-05-01T05:11:15Z
  starting request {
    "method": "HEAD",
    "castExists": true,
    "path": "/box",
    "query": {}
  }
  

Elasticsearch DEBUG: 2018-05-01T05:11:15Z
  Request complete

Index exists?  false
Index already exists?  false
Index does not exist, lets create it!
Elasticsearch DEBUG: 2018-05-01T05:11:15Z
  starting request {
    "method": "PUT",
    "path": "/box",
    "query": {}
  }
  

Elasticsearch DEBUG: 2018-05-01T05:11:16Z
  Request complete

Index created:  { acknowledged: true, shards_acknowledged: true, index: 'box' }
Index status:  { acknowledged: true, shards_acknowledged: true, index: 'box' }
Elasticsearch DEBUG: 2018-05-01T05:11:16Z
  starting request {
    "method": "HEAD",
    "castExists": true,
    "path": "/box/_mapping/files",
    "query": {}
  }
  

Elasticsearch DEBUG: 2018-05-01T05:11:16Z
  Request complete

Index type mapping exists?  false
Type mapping exsits?  false
Type mapping does not exist, lets create it!
Elasticsearch DEBUG: 2018-05-01T05:11:16Z
  starting request {
    "method": "PUT",
    "path": "/box/_mapping/files",
    "body": {
      "files": {
        "date_detection": false,
        "properties": {
          "event_id": {
            "type": "keyword"
          },
          "event_type": {
            "type": "keyword"
          },
          "created_at": {
            "type": "date",
            "format": "date_hour_minute_second"
          },
          "item_id": {
            "type": "keyword"
          },
          "item_type": {
            "type": "keyword"
          },
          "item_name": {
            "type": "keyword"
          },
          "parent_id": {
            "type": "keyword"
          },
          "parent_type": {
            "type": "keyword"
          },
          "parent_name": {
            "type": "keyword"
          },
          "created_by_id": {
            "type": "keyword"
          },
          "created_by_type": {
            "type": "keyword"
          },
          "created_by_name": {
            "type": "keyword"
          },
          "created_by_login": {
            "type": "keyword"
          },
          "content": {
            "type": "text"
          }
        }
      }
    },
    "query": {}
  }
...

```

さて、indexが一段落したら検索可能な状態にして上げる必要があります。
http://localhost:5601/ にアクセスしてindexを設定します。

Management > Indexを選択
![](https://cloud.box.com/shared/static/9jsv0dvklx3werpuv89zzzomvkoholsk.png)

Index名に適用な名前をを入れる
![](https://cloud.box.com/shared/static/2r3ldcu0tbhhmouszfuswf0p4dpt8ilo.png)

Filterにcreated_atを設定したら完了
![](https://cloud.box.com/shared/static/k6tt1hfjuq2vnnbcuol8juj9auy5oogq.png)


## 検索してみよう
ここまで設定したら、フルテキスト検索が可能になっているはず。
左上のDiscoverを選択して、Time Rangeをいじります。
デフォルトだと、15分以内にアップロードされたファイルが検索対象となっています。適宜設定しなおしてくれぃ。
![](https://cloud.box.com/shared/static/qem6fazkp32yafl4e4t7i0hmh6cnndoy.png)


## indexの調整

indexのマッピング情報はこちら：
ここに情報を追加すれば、Elasticsearch上のindex情報を追加出来る
https://github.com/kylefernandadams/box-elasticsearch/blob/master/index/type-mapping.json

event情報のパーサーはこちら:
https://github.com/kylefernandadams/box-elasticsearch/blob/master/events/events-parser.js

Index用にひっぱってくるファイルの情報はBox Representation APIを使用:
https://github.com/kylefernandadams/box-elasticsearch/blob/master/events/text-extractor.js

[text-extraction]だけをひっぱってくるので、生データよりもかなりデータ量を減らすことが可能となります。

Box Representaion APIの説明は[こちら](https://qiita.com/daichiiiiiii/items/12c2bb163fe013f400a9#box-representation-api)


# まとめ

さて、検索indexを貼れるようになったはいいものの、こういったfull textソリューションの難しいところは、検索結果のセキュリティをどうやって担保するか？
以下のことを考慮する必要があります。

1. indexとして引っ張ってくるユーザーの権限
 - どのコンテンツを引っ張ってくるのか？
 - 企業内すべてを引っ張って良いのか？特定のコンテンツだけをひけるようにするべきなのか？

2. プレゼンテーションをどうするべきか？
 - せっかくbox上でアクセス制限しているのに、full textアプリ側でなんでもかんでも見れるようにしてしまっては意味がない。
 - 誰にどのコンテンツを見せるのか?
   - box上のセキュリティ権限と合わせるのか？合わせる必要があるのか？
 - 日々アップデートされるコンテンツのセキュリティ権限をどうやって同期するのか？

Boxに限ったことではなく、どのシステムと組み合わせても上記の設計は必要になってきます。
Box + Elasticsearchを組み合わせて使用している他社のお客様事例だと、Enterprsie Eventではなく特定のフォルダのみfull text対象としているようです。
そしてKibanaにアクセス出来る人を検索対象としているboxフォルダにアクセス権がある人に絞っているそうです。

"full text検索"って聞こえは良いけど、一番重要なのはなんのためにこのシステムを開発したいのか？目的をはっきりとしておくこと。そうしないと車輪の再開発になってしまう。
