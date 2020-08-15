---
title: Rubykaigi2018 - Matz Keynoteまとめ
date: 2018-06-05 10:00:00
tags: 
 - ruby 
 - rubykaigi
 - matz
---

**Keynote講演内容まとめ：**

今更ながら、今年のRubykaigiのMatzさんのkeynoteをまとめてみた。
Noteを取り間違えた部分があるかもしれません。
間違っていたら指摘してください。直します。



# 開発において、名前付けはとても重要な作業のひとつだ。

プロジェクトの名前はとても重要だ。いまの時代Googlabilityを意識しないといけない。なぜならGooglabilityが低いとCommunityがそれを見つけられないからだ。

たとえば、一般名詞や動詞をプロジェクト名につけるのは良くないだろう。Googlabilityが低くなってしまう。

**例:**
- Go
- Swift

ただし、これらのプロジェクトは巨大な企業が支援しており、コミュニティのベースを強制的に作り上げることが可能だ。

巨大企業の支援がない場合、名前に遊びをいれると良いだろう

**一文字いじってみる：**

Streem (自画自賛)

**複数の単語をつなげる：**

Ruby on rails

**造語：**
<s>Zendesk (ここで使われたのは本当はZendeskじゃないんだけど、なんて言っていたか忘れてしまった。)</s>

tensorflow (ご指摘ありがとうございます！)

どれだけ良いプロジェクトでも、見つけてもらわないことにはコミュニティは形成されない。

# ClassやMethodの名前決めは重要

次にClassやMethodに名前をつける場合も気をつける必要がある。私は過去にこのようなMethod名をつけてしまった。

```ruby
def yield_self
  yield self
end
```

名前と挙動が全く一緒で何がしたいのか全くわからない。
このメソッドを思いついたとき違うことを思い描いていたはずだ。

もっと大きなスコープで何を処理したいのか、常に頭にいれて名前をつけるべきだ。

最近このメソッドに対して修正をCommitした。実に５年ぶりに意味のあるCommitをしたそうだ。（笑
https://github.com/ruby/ruby/commit/d53ee008911b5c3b22cff1566a9ef7e7d4cbe183



# Rubyの軌跡は「塞翁が馬」だった。

Rubyの軌跡はまさに[塞翁が馬](https://ja.wikipedia.org/wiki/%E6%95%85%E4%BA%8B#%E5%A1%9E%E7%BF%81%E3%81%8C%E9%A6%AC)といえる。

Rubyを発表した当初は、誰も見向きもしてくれない言語だった。
なんでわざわざRubyをつかう必要があるのか？

Ruby on Railsの登場以降は、多くのユーザに届くようになり、一時期はもっともホットな言語になったこともある

しかし、近年はまた下火になりつつあり、ついには「Ruby is Dead」と言われる始末。

![](https://cloud.box.com/shared/static/p2fv2yew76g4kekh7tido57q5nz3iyqo.jpg)

Rubyは遅いとか後発言語のほうがすぐれているからRubyを使う必要がないとか

毎年なんらかの理由でRubyは死んだと言われる。

Ruby is dead every year (このスライドの写真撮り忘れた。。。)

理由はそれぞれあるが、私は気にしない。

言語にはそれぞれups & downsがある。

IntelのCEOが昔このような言葉を発していた。

![](https://cloud.box.com/shared/static/8pohzmaoehgr4uud8qdmyjlc2mcs6vwh.png)

_Only the paranoid survive_ - 偏執症のみが生き残る。
病的なほど疑り深い人ほどあれこれ、試してみたくなるものだ。

我々は常にRubyを進化させる。進化に向けていろいろ取り組んでいるので決して死んでいるとは思わない。と言って締めくくった。


***
_Only the paranoid survive_ には序文があって実はこれが重要なメッセージを持っていると思っています。

Success breeds complacency. - 成功は慢心を生む。
Complacency breeds failure. - 慢心は失敗を生む。
Only the paranoid survive。 - 偏執症のみが生き残る。
![](https://cloud.box.com/shared/static/n9uaike4xqxo4z3mhv6yhozsxb359suh.jpg)


# Q&Aセッション

静的型付け言語(typescript等)が巷では流行っているが、Rubyでの型付けについてどう思うか？という質問への回答が興味深かった。
matz曰く、静的型付けのメリット(流行っている理由)は理解しているつもり。ただし、Rubyでは型宣言できるようにする予定はないと答えた。
なぜならコンパイラがもっともっと頭がよくなって行ってコンパイル時に型を予測してくれる未来が見えるから。
もしかしたら、2040には静的型付け言語はダサい、古いと言われているかもしれない。

ただし、この未来を実現するには、静的型付けの研究に投資をしなくてはならないし、rubyでも型付けについて研究しているチームがいる。

自由に記述出来ることこそがrubyのメリットであり、楽しさであることを忘れてはいけない
