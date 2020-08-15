---
title: Amazon Dash ButtonをIoTボタンにする方法
date: 2018-01-13 00:00:00
tags:
 - AmazonDashButton
 - IoT
---
#まえがき
以前Facebookにポストした、Amazon Dash ButtonをIoTボタンとして使うやりかたどうやるの？と何人かに聞かれたので備忘録含め公開

#FacebookにPostした内容

```facebook
有志がコーヒー豆を持ち寄って美味しいコーヒーを会社で淹れているんだが、コーヒーマシンのボタンを誰かがポチったことを会社全体に通知する方法がないか考えてた。
家に眠ってたamazon dashボタンを使って簡易コーヒーマシン稼働したよ！ボタン作ってみた。
ボタンを押すとSlackに通知が行く仕組み。
明日さっそく使ってみよう。
コーヒーが沸いたら通知してくれるのがベストだけどセンサー類の設定がめんぐせぇからまずはこれでよしとする。
```
![](https://dbox.box.com/shared/static/1b8l5t9vqhvdhseg937ryyfj6htijcyy.gif)

**会社のSlackはWebhook許可していなくて断念したんだけどね...**
#ぼやっとしたAmazon Dash Buttonの仕組み
Dash Buttonを買ったのはいいが、うちの嫁が配達屋さんの仕事が増えてかわいそうだから使っちゃ駄目だということで、エリエールとスコッティのDash Buttonがうちに転がっていた。

Amazon Dashボタンが出たばっかりの頃、ボタンを押すと同一ネットワークにARPをかけているというのをぼんやり読んだことを覚えていた。
これってARPを拾ったらIoTボタンになるんじゃね？って思ったのがきっかけで作りました。

- Amazon Dash Buttonは、スマホのAmazonアプリと連動している。
- AmazonアプリにDash Buttonを登録すると、Wifi接続情報がDash Buttonに登録される。
- Amazon Dash Buttonに商品登録する手順の手前までで止めることで、ボタンを押しても商品が注文されず、IoTボタンになる。

##code

```ruby
#!/usr/bin/env ruby

#packet filtering用のライブラリ
require "packetfu"
#WebhookにPostするために使うRestライブラリ
require "rest-client"
require 'json'

include PacketFu
FILTER="arp and arp[7]==1"

dev = ARGV[0]

#パケットキャプチャ開始
cap = Capture.new(:iface=>dev, :start=>true,
                  :filter=>FILTER)

cap.stream.each do |pkt|
  next unless ARPPacket.can_parse?(pkt)
  tstamp  = sprintf("%.6f",Time.new.to_f)
  arpreq  = ARPPacket.parse(pkt)
  src_mac = EthHeader.str2mac(arpreq.eth_src)
  src_ip  = arpreq.arp_src_ip_readable
  puts "#{tstamp},#{src_mac},#{src_ip}"
  　if src_mac == 'xx:x:xx:xx:xx:xx' 
       payload = {channel: "#コーヒー", "username": "Coffebot", text: "<!here> コーヒー淹れたよ！たぶん10分後にコーヒーポットいっぱいなるよ", icon_emoji: ":coffee:"}
       r = RestClient.post 
　　　　#Slack WebhookにPostする
　　　　'https://hooks.slack.com/services/randomstring/randomstring', payload.to_json
 　　end
```

いつかDash ButtonとBoxを組み合わせられたら面白いかも。

#参考
[パケットキャプチャ](https://qiita.com/akito1986/items/f40ce9e19dc7524cd691)
[Slack webhook](https://api.slack.com/incoming-webhooks)

