---
title: RubyのメソッドにConstできない
tags:  
 - ruby
 - const
---

# メソッド内で変数宣言できない！？

意外なところで新発見をしてしまうことって結構ありますよね。
rubyでとあるプログラムを書いていて初めて以下のエラーになりました。

```ruby
abbyy.rb:60: dynamic constant assignment
    APPLICATION_ID = CGI.escape("chomericchi")
```

調べてみたら、どうやらrubyではメソッド内にConstが指定できない仕様である。
http://rubylearning.com/satishtalim/ruby_constants.html

そんな制限があるんですね。。。以外でした。
いやそれより、いままでこのエラーに遭遇しなかった自分に驚くわ。
まぁ、たしかにメソッド内に定数を書くのはよくないよね。
メソッドの中に書いてしまうと、メソッドを呼び出すたびにOjectが変わってしまうから本当の意味で定数ではなくなる。

```irb
irb(main):014:0> A = "test"
=> "test"
rb(main):015:0> A.object_id
=> 70139672672700
irb(main):016:0> A = "test"
(irb):16: warning: already initialized constant A
(irb):14: warning: previous definition of A was here
=> "test"
irb(main):17:0> A.object_id
=> 70139659793140
```