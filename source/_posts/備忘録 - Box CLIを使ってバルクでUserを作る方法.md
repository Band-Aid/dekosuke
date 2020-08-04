---
title: 備忘録：Box CLIを使ってバルクでUserを作る方法
date: 2018-05-09 00:00:00
tags: 
- box
- BoxCLI
---
備忘録：Box CLIを使ってバルクでUSerを作る方法
# 備忘録

Box CLIにCSVを渡して一気にUserを作る方法。

box users create --bulk-file-path ./users.csv

この、CSVの作り方がちょっと特殊だったので、忘れないように書いておく。


## CSVの書き方

[Box CLIのヘルプ記事](https://developer.box.com/docs/box-cli)を読んでも、Note: CSV files are also supported for bulk actions, but require special CSV templates.

としか書いてなくて、役にたたない。

[ソース](https://github.com/box/boxcli/blob/master/BoxCLI/CommandUtilities/CsvModels/BoxUserRequestMap.cs)を探ってみた。

ふむふむ。
Name,Login,IsPlatformAccessOnly,Status,Address,Phone,JobTitle,Language,SpaceAmount,IsSyncEnabled,IsExemptFromDeviceLimits,IsExemptFromLoginVerification,IsPasswordResetRequired,CanSeeManagedUsersをヘッダーに入れる必要があるそうな。


こんな感じのCSVが必要になります。
サンプルファイル：https://cloud.box.com/s/dtgmar034n6nh4f3lmdprb0guyx99dj2

~~*1 ID,Name,Loginは最低限値の入力が必須なフィールドです。IDは何を入れても良いです。User作るときIDは指定出来ないので、多分[バグ](http://github.com/box/boxcli/issues/75)っぽい。~~←直った！

*2 各種フィールドの詳細は[box APIドキュメント](https://developer.box.com/v2.0/reference#create-an-enterprise-user)参照ください。

*3 言語コードを指定する場合は[こちらを参考にしてください](https://cloud.box.com/s/pmiwnto70ts6o036g8n1ec31o37ctmhv)

# 備忘録終わり


なんか全体的にデジャブ感のあるエントリーになったなぁ。(白目)

[備忘録：Box CLIを使ってバルクでGroupを作る方法]()
