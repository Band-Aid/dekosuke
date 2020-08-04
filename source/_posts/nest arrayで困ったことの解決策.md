---
title: nest arrayで困ったことの解決策
tags: 
 - javascript
 - array
---

# Nested Javascript Array

ネストされたArrayを処理していて困ったので、備忘録の意味を込めて書き起こし

## 問題
こういうArrayデータがあり、singleLineのデータを取り出そうとしたらエラーになった。

```javascript
var a = [ 
  { boxedCharacters: { text: '１０４００４３' } },
  { singleLine: { text: '東京都中央区日本橋新古町' } },
  { singleLine: { text: 'この度は、数多くのお店の中から当店を'} },
  { singleLine: { text: 'お選びご来店頂きまして誠にありがとう'} }
]

a.forEach(function(value){
      console.log(value.singleLine.text)
      })

>>Uncaught TypeError: Cannot read property 'text' of undefined
    at <anonymous>:9:36
    at Array.forEach (<anonymous>)
    at <anonymous>:8:4
```

最初は、boxedCharactersというkeyがあるためsingleLineだけを取り出そうとしているからエラーになったかと思ったが、.textにアクセスしなければ、問題なく各種オブジェクトが取り出せたので、たぶん同じレベルに同名のKeyがあるせいだと気づく。

```javascript
 a.forEach(function(value){
      console.log(value.singleLine)
      })

 >>
 {text: "東京都中央区日本橋新古町"}
 {text: "この度は、数多くのお店の中から当店を"}
 {text: "お選びご来店頂きまして誠にありがとう"}     
```

## 解決策

[Stackoverflow](https://stackoverflow.com/questions/50066624/how-to-access-nested-json-array)のguruたちに聞いてみた。
質問してからものの、数分で的確な回答がもらえる。すげぇ。
.mapをつかって要素一個ずつ取り出し、singleLineがあるかどうか、そして.text要素を持っているか確認する。もし、違うのであればfilterして取り除く。とても勉強になりました。

```javascript
    const a = [ 
      { boxedCharacters: { text: '１０４００４３' } },
      { singleLine: { text: '東京都中央区日本橋新古町' } },
      { singleLine: { text: 'この度は、数多くのお店の中から当店を'} },
      { singleLine: { text: 'お選びご来店頂きまして誠にありがとう'} }
    ];

    const getSingleLine = o => (o&&o.singleLine);
    const getText = o => (o&&o.text);
    const getSingleLineText = o => getText(getSingleLine(o))

    console.log(
      a.map(getSingleLineText)
      .filter(x=>!!x)//remove undefined
    )

>> ["東京都中央区日本橋新古町", "この度は、数多くのお店の中から当店を", "お選びご来店頂きまして誠にありがとう"]
```
