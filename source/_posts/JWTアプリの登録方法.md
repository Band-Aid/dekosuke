---
title: JWTアプリの登録方法
date: 2018-09-16 15:50:10
tags: 
- box
- JWT
---
# JWTアプリケーションの登録方法を記載します。

JWTアプリケーションの登録は、メイン管理者で行って頂く必要がございます。

## 1. 管理コンソール > エンタプライズコンソール > アプリ

![](https://cloud.box.com/shared/static/lzolowz1eox6xj4q5ug1d0yinyi4tawt.png)

## 2. 中程のカスタムアプリケーション項目の新しいアプリケーションを承認を押下します。

(下記の例ではすでにいくつかアプリケーションが登録されていますが、何も登録がなければ空欄になっています。)

![](https://cloud.box.com/shared/static/h04dgvshdwd4s7gnahy8togd0zzd4itd.png)

## 3. アプリケーションのAPI Keyを入力します。

[こちら](https://qiita.com/daichiiiiiii/items/d54b856ebaf9f00c528b)の資料で作成するアプリケーションのClient ID(API Key) を入力します。　

![](https://cloud.box.com/shared/static/r5656t9jg9tcb1gmf3xh1nl7443oejrv.png)

## 4. アプリケーションのスコープを承認します。

以下の情報は一例になります。アプリケーションに付与しているアプリによって以下内容は異なります。

![](https://cloud.box.com/shared/static/1xpmhwvlejnrtn1yqena0tvgkff0bh1q.png)

## 5. 承認後、カスタムアプリケーション一覧にアプリ名が表示されているはずです。

[こちら](https://qiita.com/daichiiiiiii/items/d54b856ebaf9f00c528b)で作成したJWTを使って、Box APIが使用できるようになります。

![](https://cloud.box.com/shared/static/odnwc1x6a0ybtrapmw1cwo3s1vultmsp.png)
