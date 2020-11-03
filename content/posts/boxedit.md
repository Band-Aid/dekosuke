---
title: "Boxedit"
date: 2020-11-03T17:24:03+09:00
draft: false
---

# Box Editの仕組み

Boxって魔法のアプリようだ。BoxのWebアプリ上からローカルのアプリが開くなんてあら、不思議

今日はその仕組みを解析してみようと思う。

Box Editをインストールすると、２種類 (ほんとは３種類だけど本稿とは関係ない機能なので省略) のプログラムがインストールされる。

Box Edit.exeとBox Local Com Server.exe

Box Edit.exeはBoxからファイルをとってきたり、編集したファイルをアップロードしたりする。

Box Local Com Server.exeは、ブラウザから飛んでくる指示を受け取って開くべきファイル情報をBox Editに受け渡すmiddle manの役割を果たしています。名前の通り、ローカルにあるCommunication Serverというわけだ。

やりとりのフローをスーパー雑にまとめるとこんな感じ

以下四角はパソコンの中をイメージしている。絵心ないのは許して。

 ![](https://dbox.box.com/shared/static/cse6asea0fg5yuc1zppvur0397clpxmb.jpg)



## Box Webアプリからの呼び出し

Box Webアプリからどうやってローカルのアプリを呼び出しているのか見ていこう。

まず、Boxにログインすると次のURLにリクエストが飛んでいることがわかる。

http://127.0.0.1:17223/status

これは、Box Editがマシンにインストールされているかどうかのチェックをするために使われている

Local Com Serverからレスポンスが返ってくればインストールされているということなる。

> {"installed":true,"running":true,"os":"Windows","version":"4.11.0.822","mixed_mode":false}

返事がなければ、インストールされていないので、編集ボタン押下時にBox Editをインストールするようユーザーを促す。

上記URLを直接たたいてみると、以下のようなエラーが返ってきているかと思う。

> {"response_type":"error","status":403,"message":"Missing Origin Header"}

Box Local Com Serverは `*.box.com` 以外からのリクエストを受け付けないようになってる。変なサイトからリクエストを受け付けない安心設計というわけだ



次は、編集ボタンを押したあと、どのような仕組みなっているか見ていく。

![](https://dbox.box.com/shared/static/fv88qbgk8wxqbb6hxbz4v1rqe598vywp.jpg)





Powerpointで開くボタンを押すと、localhost:17223/というホストに対してリクエストが飛んでいる。

これはローカルにインストールされたBox Editのプログラムにリクエストがかかっている。

![](https://dbox.box.com/shared/static/ah71ekiem9jqs70itspxj7yjuv4pl2ud.jpg)

リクエストのbodyを見てみよう。

```json
{"command_type":"launch_application","file_id":"737291821870","auth_token":"9j97hygd7y1ka1mu22vti28pee4wsehk","auth_code":"CB3JmpYzZ0Y4CPt5pzsuUwVfWFnF2jcl","key_id":"1595l1jr","efile_id":"oarl1ipKAfJ35HtD5Xtlew==","eauth_code":"iO4vGpT3ZCZOPTxrf/vpCkhJDfrQ48tKSy5PoHc5AlvGQJIFXovkVFg6nRXSOKx8","browser_type":"Chrome","token_scope":"folder","timestamp":1604392354515}
```

`command_type: launch_application` とあるように、アプリ起動指示のようだ。また、どのファイルなのか？ `file_id: 737291821870` に対して支持を出しているというのがわかる。

そのあと、auth_codeやKey_idという何やらBox access tokenに必要そうなデータが含まれている。

この値は、File_idに指定されたファイルをダウンロードするために使う。後述

また、引き換えられるTokenは `file_id`の親フォルダのみにDownscope*されているようだ。

> token_scope: folder

つまりなんでもできるTokenじゃなくて、ある程度権限が絞られたTokenというわけだ。
***Downscope Tokenの詳細については、[こちら](https://ja.d**eveloper.box.com/guides/authentication/access-tokens/downscope/)

さて、このリクエストを受け取ったLocal Com Serverはどんなレスポンスをブラウザに返しているのだろうか。

```json
{"response_type":"command","success":"true"}
```

`Ok,わかった。指示を受け取ったよ。`  そんなレスポンスを返している。

さて、先ほど受け取った指示はどのように処理しているのだろうか。



ここから先のやりとりはBox Local ComとBox Editのログに記録されている

ログは以下のフォルダに記録されている。(windows&Box Edit.exe*インストーラーを使った場合以下のパスに保存されている

***.msiインストーラーの場合は、Box Local Com Serverだけprogram files配下に置かれる。でもその話はまた今度**

> %localappdata%\Local\Box\Box Edit\

> %localappdata%\Local\Box\Box Local Com Server\



先ほどのリクエストを受け取ったあと何が起きているのかを見るには、Local Com Serverのログを見てみよう

9行目以降にブラウザからリクエスト受け取ったあとの処理となっている。

ブラウザからリクエストを受け取り、受け取った内容をBox Edit.exeに受け渡している。

```json
[2020-11-03 17:54:51,924] [31   ] [INFO ] [] - Handling application message
[2020-11-03 17:54:51,928] [15   ] [DEBUG] [] - Sending {
  "command_type": "launch_application",
  "file_id": "737291821870",
  "auth_token": "y0yiswbgagg0prkvuoicz9j5b7wcmstg",
  "auth_code": "VOpj6swg9s1uJ19wIe8aaZycQI5tDPNH",
  "key_id": "1595l1jr",
  "efile_id": "oarl1ipKAfJ35HtD5Xtlew==",
  "eauth_code": "LOwSWCv9vAJS/AjRZx+IIAUsQJK2qgTmTNDtrrVRtu4CvEQ06+RAep4WuHYLyg+3",
  "browser_type": "Chrome",
  "token_scope": "folder",
  "timestamp": 1604393691860,
  "com-id": "bgp-2a8ea140",
  "require_consent": false,
  "origin": "https://dbox.app.box.com"
} to application boxedit through named pipe box_edit_pipe
```

今度は、Box Edit.exeの内容を見てみよう。

Box Edit側のログには、Box Local Com Server 側から受け渡された値が記載されている。

この受け渡された情報を使ってBox Access Tokenと交換する。Box Access Tokenを使ってBoxの情報にアクセスする。

Access Tokenの発行の流れは、この辺りを[参照]([OAuth 2.0認証 - Box開発者向けドキュメントポータル](https://ja.developer.box.com/guides/authentication/oauth2/))

```json
[2020-11-03 17:54:51,933] [8    ] [DEBUG] [] - PipeServer - Received message through box_edit_pipe: {
  "command_type": "launch_application",
  "file_id": "737291821870",
  "auth_token": "y0yiswbgagg0prkvuoicz9j5b7wcmstg",
  "auth_code": "VOpj6swg9s1uJ19wIe8aaZycQI5tDPNH",
  "key_id": "1595l1jr",
  "efile_id": "oarl1ipKAfJ35HtD5Xtlew==",
  "eauth_code": "LOwSWCv9vAJS/AjRZx+IIAUsQJK2qgTmTNDtrrVRtu4CvEQ06+RAep4WuHYLyg+3",
  "browser_type": "Chrome",
  "token_scope": "folder",
  "timestamp": 1604393691860,
  "com-id": "bgp-2a8ea140",
  "require_consent": false,
  "origin": "https://dbox.app.box.com"
}
```

発行したBox Access Tokenとファイルは、以下フォルダに格納されている。

俗にキャッシュフォルダと呼ばれるフォルダだ。

> %localappdata%\Local\Box\Box Edit\Documents

英数時のランダムなフォルダができていると思う。その中にそれぞれのファイルと対応するBox Access Tokenが保存されている。

Access Tokenは `.metadata` という隠しフォルダに保存されている。この `.metadata`　フォルダには様々な情報が入っている。

落としてきたファイルID、ファイルの名前、ファイルの最終アクセス日等*****など

***Box Editは最終編集日よりある一定日以上アクセスされていないと、ファイルが削除される。この規定日はコロコロ変わるので最新の情報はBoxに問い合わせて聞いてほしい。**



`.metadata` フォルダに保存されているAccess Tokenは前述したようにDownscopeされている。落としてきたファイルと同じフォルダ内のフォルダに対しては操作できるがそのほかのフォルダの操作できないようなっている。

このAccess Tokenが盗まれたとしても情報流出は最低限に抑えられるようになっている。(もっともこのAccess Tokenが盗まれるようなPCへの侵入事案が発生したら、もっと心配すべき事態になっていると思うが。。。)

さてファイルをダウンロードし、キャッシュフォルダにファイルを保存したあと、Box Edit.exeは定期的(1秒に1回程度)キャッシュフォルダを巡回している。先ほどのmetadataファイルと比較して、変更あったファイルを検知し、新しいバージョンとしてBoxにアップロードを行う。

ちなみWindowsには、ファイル変更検知を監視するためのAPIが用意されている。何故ファイルの変更検知ではなく、１秒に１回の巡回を行っているのか？疑問に思った人もいると思う。

以前中の人に聞いたところ、変更検知はPCの処理が重くなると通知がきちんと送られない不具合があるそうだ。

その不具合を回避するのに巡回することをWindowsの開発者に勧められたとか。



終わり。




# おまけ


## Box Editが開くべきファイルをどうやって認識しているのか。

ファイルをクリックしたり、プレビューを開くとこんなリクエストがブラウザからLocal Com Serverに飛ぶ

GET

> http://127.0.0.1:17223/application_request?application=BoxEdit&com=bgp-2a8ea140&timeout=4&ms=1604394153133

Bodyには

> {"request_type":"get_default_application","extension":["pptx"]}



これは何をしているかというと、拡張子pptxを開けるアプリケーションはありますかー？という問いかけを行っている

リクエストを受け取ったBox Local Com ServerはBox Edit.exeにさらにこの拡張子を受け渡す

Box EditはWindowsのレジストリを見に行く。pptxに結びついている編集動詞を参照する

Windowsはアプリケーションを"開く"際、様々な動詞を使用してファイルを開く

たとえば、.csvファイルをダブルクリックして開いてみましょう。Excelがインストールされている人は、Excelで開いたと思う

.csvファイルを右クリックして、"編集"オプションを選択してみよう

あれ、今度はnotepadアプリで開きましたね？これは、Windowsは標準で編集動詞にnotepadが結びついているからだ

Box上でcsvファイルを編集しようとするとnotepadアプリが開くのはこのせい

[ファイル名拡張子に対する動詞の登録 - Visual Studio | Microsoft Docs](https://docs.microsoft.com/ja-jp/visualstudio/extensibility/registering-verbs-for-file-name-extensions?view=vs-2019)



**閑話休題：**

先ほどBox Local Com Serverのログを振り返ってみましょう。7行目のReceived以降を見ると

 Box Edit からPPTXに結びつくアプリはpptxだというのが返ってきたと言っているのがわかる。

> [2020-11-03 17:54:47,159] [31   ] [INFO ] [] - Received {"response_type":"get_default_application","default_application_name":{"pptx":"PowerPoint"}} from application boxedit

