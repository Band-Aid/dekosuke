---
title: Box Hackathon Jan 2018
date: 2018-01-30 00:00:00s
tags: 
 - box 
 - hackathon
---
# Box Hackathon

先日、今年最初のbox hackathonが社内で開催されました。半年に一回boxでは、社内hackathonが開催されます。
24時間以内に面白いhackを作って1分以内のビデオにまとめてboxにアップロードします。
全社員による投票が行われます。

一応主催者からお題は出されますが、お題以外の賞もあります

```
- 投票が多かったで賞
- boxerっぽいで賞 (box社員の事をboxerと呼びます)
- Aaron(CEO)が気に入ったで賞
```

などなど
hackは必ずしもお題に沿った内容である必要はなく、かつ、テクニカルな内容である必要もありません。
オフィス環境の改善hackでもOKです。

**たとえば、**

```
廊下の曲がり角で良く人がぶつかるので、鏡を置いてみた！とか
電話ブースの使用状況が分かるように、使用中の紙札を作ってみた！とか
```

テクニカルな内容で言うと、hakathonから生まれたアイディアが実際にboxの製品に組み込まれたりしています。
どれとは言いませんが、かなりの製品がhackathonから生まれたアイディアだったりします。

# ほんで、お前は何出したの？

こんなん出してみました。
音出し推奨！
動画：
{% raw %}
   <iframe src="https://cloud.app.box.com/embed/s/tlj5rwjwur6p0jtzw7sl0bke8vycsyrp" width="600" height="400" frameborder="0" allowfullscreen webkitallowfullscreen msallowfullscreen></iframe> 
{% endraw %}


＊これ以外にも真面目なものも出しましたよ！


## 仕組み

以下のアイディアパクりました。ありがとうございます。

[DevTools を開いたら人類滅亡](
https://qiita.com/diescake/items/b25791eb7750c775e72f)

iframeにbox.comを入れて、DevToolsの有無を確認しています。
DevTools開いたら人類滅亡します。5分で作りましたw


# え？それってCross domain callにならないの？大丈夫？

boxでは、XSS対策をしています。心配いりません。
この動画以上の変なことしようとすると、エラーになります。

# boxは楽しい会社です。

この作品それなりに得票つきましたw
でも、残念ながら入賞しませんでした。次のhackathonは入賞目指して頑張ります。
