---
title: Box APIをPostmanで叩いてみよう
tags: 
- box 
- Postman 
- boxapi
---
Box APIを試してみたくてもCurlでURLを叩くのがめんどくさい、どういうエンドポイントがあるのかわからない
そんなときは、Postmanを使って簡単にエンドポイントに対してコールを試してみることが出来ます。

# Postmanとは
postmanはWebサービスAPIを簡単にテストできるツールになります。
Box APIをpostmanを使って呼び出してみましょう

### このドキュメントでは以下が説明されます。
- Postmanの設定手順
- Box API初期設定手順
- Postmanを使ってBox APIを呼び出す

## 事前準備：
**Postmanを使用するのに必要な環境：**
###①[Postmanのネイティブクライアント](https://www.getpostman.com/) をインストール
(Chrome AppsがEoLを迎えるためネイティブアプリの導入を推奨
###②Boxアカウント 
<font color="red">(以下の設定を行うとBox API経由でデータにアクセス可能となります。テスト環境での検証を強く推奨いたします。)</font>

## Postmanの設定
### ①Postmanをホームページからダウンロード、インストール
![](https://cloud.box.com/shared/static/jy9gb0wlqz46wk8baj7k63z35f9nk3k6.png)

### ②インストール後、Postmanを起動します。
### ③postman起動後、アカウントサインインを求められますが、SKIPできます。
![](https://cloud.box.com/shared/static/5t4ucffnqmdoq1hc2kdm3jdqv89isre6.png)

### ④Box APIエンドポイントのインストールを行います
下記リンクをクリックし、変遷後のURLをコピーします
https://cloud.app.box.com/v/postman-box-content
![](https://cloud.box.com/shared/static/3uiq4mo35hgp95mnxalug8qcucj3ocmt.png)
![](https://cloud.box.com/shared/static/cynylqvp6gskd7u8otkgr3y6ny2q07m1.png)

### ⑤Postmanを起動し、IMPORTボタンを押下し、IMPORT FROM LINK設定にURLを入力します
![](https://cloud.box.com/shared/static/v9oqegvcx9f8f483ioa22hl5mksagj9j.png)

### ⑥Collectionタブを選択すると、Box APIエンドポイントがインストールされているはずです。
![](https://cloud.box.com/shared/static/yn9p201a0monzrr6tc5vevgeztmhi251.png)


# Box APIの設定
Developer Tokenの取得方法は以下のドキュメントを参照ください。
https://cloud.box.com/s/4inscvj7o04jh1u0kym7ktkawcwwcrth
<font color="red">＊以後、API経由でデータが取得出来るようになります。
まずは、テスト環境でのログインを推奨いたします。</font>

### ①上記で取得した開発者トークンをコピーします
### ②postmanアプリにの戻り"Usersフォルダ" > Get the Current User's informationを選択します
![](https://cloud.box.com/shared/static/nvdrbtdo5opvr5v4ck2npdqww62yvu0t.png)

### ③Headerタブを選択し、Authorizationキーの値にアクセストークンを挿入します。
![](https://cloud.box.com/shared/static/kmulctjsjmiigh3b0l22vzj5cmp9uwxt.png)

例；
![](https://cloud.box.com/shared/static/szib1uj4qg50mrybpcre312t2eizb98j.png)


### ④Body欄にJSON形式でユーザ情報が返ってきているはずです。
![](https://cloud.box.com/shared/static/f0sj9euj6jxkpg7jw9du8nt0xz5ueqr1.png)

### ⑤他のAPIコールも呼び出し方は基本的に同じです。
APIエンドポイントによっては管理権限がないと呼び出せないものもあります。
また、データの呼び出し方の作法を守らないと呼べないものもあります。
各種エンドポイントの呼び出し方法はBox APIドキュメントをご参照ください
日本語：https://ja.developer.box.com/reference
英語：https://developer.box.com/reference

動画
![](https://cloud.box.com/shared/static/yf83av8lc93c4cmq17xizdfzgst7eopg.gif)


