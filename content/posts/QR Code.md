---
title: box共有リンクをQR Codeにする
tags: 
 - box 
 - web統合
 - QR
---

# 共有リンクをQR Codeにする

Boxのユースケースのひとつに、QR Code化した共有リンクをプリントアウトして、あっちこっちに貼っておくという使い方がある。

**<font color='red'>これ、結構便利なんですよ</font>**

たとえば、オフィスのプリンターやプロジェクターの操作マニュアルをBoxにUPLOADしておき、そのファイルの共有リンクをQR Code化して、各機器に貼っつけておけば、使い方がわからなくなったらQR Codeをスキャンして、その場でドキュメントを確認出来る。

# でも、QR Code化する作業が結構めんどくさい

共有リンクをQR Code化しようとすると、QR Codeジェネレーターを導入する必要がある。
手軽にQR Codeを作れるのは、QR Codeジェネレーターサイトを使って生成する方法だが、この方法だと実行ステップが多いうえに、ウェブ上のサービスを使うと、共有リンクがウェブサイト側に渡ってしまう懸念がある。

1. 共有リンクを作る
2. リンクをコピー
3. QR Codeジェネレーターサイトをググる
4. 共有リンクをペースト
5. 生成されたQR　Codeを保存

ステップを減らしつつ、セキュリティを担保する方法はないか考えてみた。

# web統合使って、直接自前のサーバーでQR Codeを生成すればいいんじゃね？

また、[Web統合](https://qiita.com/daichiiiiiii/items/12c2bb163fe013f400a9)かよと言われるかもしれないが、またです。
Web統合を使って共有リンクを自前のサーバーに送ってQR Codeを返すコードを実装してみた。


## 事前準備

今回も例によってRuby SDKを使います。次回はnode.jsとか.netとか使ってみようかしら。
QR Codeの生成ライブラリに[rqrcode](https://github.com/whomwah/rqrcode)を使いました

```ruby
require 'sinatra'
require 'boxr'
require 'rqrcode'
require 'rqrcode_png'

post '/qr' do
  #Box Web統合からFile IDとAuth Codeを送ってくる用に予め設定
  fileid = params[:fileid]
  code = params[:code]
  #Access Token取得
  atoken = getToken(code)
    begin
        client = Boxr::Client.new(atoken)
        #共有リンクを生成
        @sharedLink = client.create_shared_link_for_file(fileid).shared_link.url
        #共有リンクからQRCodeを生成
        qr = RQRCode::QRCode.new( @sharedLink, :size => 10, :level => :h )
        #QR CodeのPNGファイルを作る
        png = qr.as_png(size: 500)
        #data url - こいつをhtml viewに返す
        @image = png.to_data_url
    rescue Exception => e
        return 'バグったよー。困ったね。'
    end
    #作ったQR CodeをHTMLで返す。ちなみにBox Web統合にサイトを返す場合、X-frames Optionの設定配慮する必要があります。
  return erb :home 
end
```

## 出来上がったデモがこちら

![](https://cloud.box.com/shared/static/gt0607g99cgoazzdnzuh58m96jss6y8w.gif)

## 一口メモ

iOS 11から標準カメラアプリにQR Codeスキャナーが実装されました。
カメラアプリを起動して、QR CodeにかざすだけでQRを認識してくれます。

![カメラをかざすだけでQRコードが開けるデモ動画](https://cloud.box.com/s/w6hhl1ufj2dqkfqriyium3qrmm3g91g9)