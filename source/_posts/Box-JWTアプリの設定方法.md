---
title: Box JWTアプリの設定方法
date: 2018-09-16 15:52:06
tags:
- box
- JWT
---
# JWTアプリの登録方法について説明したいと思います。

## 事前準備：

以下サイトからBox開発者コンソールにアクセス下さい。

https://app.box.com/developers/console

上記サイトにアクセスできない場合は、https://developer.box.comのConsole ボタンからログインして下さい。

![](https://cloud.box.com/shared/static/klcpz3gyse3bj8njcp2ponhqiglmgcek.png)

# 設定方法

## 1.  Box Developer Consoleにアクセスし、アプリの新規作成を押下します

![](https://cloud.box.com/shared/static/ah6lf1s9m0meajb56pms6t5iu0c2bfjv.png)

## 2. カスタムアプリケーションを選択し、次へ

![](https://cloud.box.com/shared/static/2rkz381rlrmzodez87wlrpotebfpp7e7.png)

## 3.  JWT認証を選択し、アプリに名前をつけます。

英語でかつ、Box全体で一意である必要がありますので、ユニークな名前を入力下さい。

![](https://cloud.box.com/shared/static/3fnzr094hls8u0p63bw1009tipv1wa9r.png)

## 4.  JWTアプリの設定

OAuth資格情報欄のクライアントIDがアプリケーションを識別するための一意のIDになります。
API Keyとも呼ばれます。

![](https://cloud.box.com/shared/static/t1904yjeys8zm7plut1b65dz21nb0bi0.png)

![](https://cloud.box.com/shared/static/benfc2um27axcl0lex9wz424qlu9h6ei.png)

![](https://cloud.box.com/shared/static/j5zr9hrzyuiy7yagzo38v47mp5t03nio.png)

## 5. 公開キーの登録

もし、Boxが自動生成するキーペアでよろしければ、右側のボタン(公開/秘密キーペアを生成)を押下下さい。

_独自のキー生成は本資料の一番下に記載致します。_

**公開/秘密キーペア**を生成ボタンを押下すると、自動的にキーペアを含んだJSONファイルが生成されます。

こちらのJSONファイルに含まれる情報を使用して、Box APIでアクセスできます。

_関係者以外にはこのファイルの共有しないようお気をつけて共有下さい_

**JWTアプリを登録するだけでは、JWTアプリはまだ使えせません。**

Boxテナントに結びつけをする必要があります。[登録方法](https://qiita.com/daichiiiiiii/items/d040babb9e990f682d8a)


# 付録：独自キーペアの作り方

openssl等を使用して、独自のKey/Pairを生成します。

以下にサンプルコマンドを添付します。

## まず、秘密鍵を生成します。途中パスワードを求められますので、入力下さい。

`openssl genrsa -des3 -out private.pem 2048`

## 上で作った秘密鍵を使って、公開鍵を作ります。

`openssl rsa -in private.pem -outform PEM -pubout -out public.pem`

生成された公開鍵をテキストエディタ等で開き、全てコピーします。

Boxの開発者コンソールに戻り、公開キーを追加を選択し、コピーした公開鍵をウィンドウに貼り付けます。

![](https://cloud.box.com/shared/static/6pr08vq5j6k3z1qivv3f11la2dofmt76.png)

![](https://cloud.box.com/shared/static/u4anvaztio9kioxji31gz4cdyl68idku.png)

![](https://cloud.box.com/shared/static/8lcqyavey7fw1nqj3qpi628q5unl466u.png)

![](https://cloud.box.com/shared/static/c8gozwcfojgdzbeeag0xxzkeo7pc28bv.png)


画面一番下のJWTのJSONをダウンロードし、テキストエディタ等で開きます。

それぞれのパラメーターに適切な値を入力して、アプリにインポートして下さい。

![](https://cloud.box.com/shared/static/69tifvgxm31sr16sojnfok71lllgyvwr.png)
