---
title: HexoとFirebaseのセットアップ
date: 2018-06-09 19:00:00
tags: 
 - hexo
 - Firebase
 - qiita
---

# Qiitaのメリット、デメリット

Qiitaは開発に関する記事がたくさんあり、開発者コミュニティの交流を促進する素晴らしいプラットフォームである。
なにより、Markdown Editorが他社サービスに群を抜いて優秀。

いっぽうで、開発者による開発者コミュニティのためのプラットフォームであるがゆえ、技術に関することの以外の記事を書いては行けない。
目的には賛同しているし、微力ながらコミュニティに貢献できるよう技術的な内容を投稿してきたつもり。。。

でも。。。たまには技術に関係ことも書きたい。
近所で見たかわいい猫のこととか、日々のくだらない _技術に関係ない_ ことも書きたい。。。

## プラットフォームの移行を考え始める

プラットフォームを移行するにもいくつか絶対条件が自分のなかにあった

1. Markdown Editorが優秀なこと
2. 投稿内容に制限がかからないこと
3. お金がかからないこと(かかっても安いこと)
4. 運用メンテを極力考えなくてもいいこと

ちょっと[ググって](https://techracho.bpsinc.jp/hachi8833/2016_12_05/30340)見たところ、結構良さそうとこもあるみたいけど、ランニングコストがかかってしまう。

サービスとしてのMarkdownサイトはお金がかかるってことで、自分でホストするオプションを探し始める。

opensource系のMarkdownプロジェクトを調べていたところ、nodejsベースのツール[Hexo](https://hexo.io/)を見つけた

かなり軽量でプラグインで拡張もできてなかなかいいじゃない。よし、次はホスティングをどうするか考えなくては。

最近同僚がGCPのFree Tierに[HackMD](https://hackmd.io/)ホストしたという話を聞いたので、Googleのホスティングプランを調べていたところFirebaseにたどり着く

無料で1GBのストレージ容量に、月間10GBの転送量まで使える。全然事足りる。

https://firebase.google.com/pricing/?hl=ja


善は急げっちゅうことでおもむろにfirebaseのSparkインスタンスを立ち上げて実装に入る

# 設定

## Firebase CLIの設定**

FirebaseをCLIで操作するためのGoogle純正ツールを入れる

`npm install -g firebase-tools`

つぎにFirebaseの認証を通す。ブラウザが立ち上がるので、認証を通す

`firebase login`

## Hexoのセットアップ

Hexo入れるよ
`npm install -g hexo-cli`

サイトのレポジトリを作るよ
`mkdir mogemogehogehoge`

Hexo初期するぜ
`hexo init ./mogemogehogehoge`

必要コンポーネントを初期化
`npm init`

Firebaseを初期化
`firebase init`

どうやってホストしたいのかいろいろ聞いてくる。
1. Hostingを選択
2. さっき立ち上げたSparkプランがついて既存プロジェクトと結びつける
3. publicディレクトリを公開用ディレクトリとして使う
4. SPAはつかわないので、`N`
5. publicを上書きしません `N`

ローカルでサイトがうまく動くかどうかテストするために、
`firebase serve`を実行。うまくいけば、localhost:5000でサイトが実行できるはず

### 設定ファイルをいじる

hexoサイトのディレクトリに_config.ymlファイルがある。
こいつをいじるとサイト名とか、フォルダの命名ルールが変えられる。

新規ポストをつくるには`hexo new "My New Post"`と入れる

そうすると、タイトル.mdファイルが /source/_postsフォルダ配下に作られる
こいつを編集し終わったらビルドする必要がある。
Firebase用のリソースをつくるためにdeployオプションもつける
`hexo generate -d`

これを行うと、/public/フォルダにエントリーが作られる

## 最後にFirebaseに変更を反映する

`firebase deploy`

うまくいけばサイトが立ち上がっているはず
https://dbox-blog.firebaseapp.com/

サイトのレイアウトと記事の移行はこれからだけど、いまのところなかなかいい感じ。

今後は、上記サイトをメインでいろいろ記事を書いていこうと思いますー。
