---
title: 備忘録：Box CLIを使ってバルクでgroupを作る方法
date: 2018-05-08 00:00:00
tags: 
 - BoxCLI 
 - box
---
# 備忘録

Box CLIにCSVを渡して一気にGroupを作る方法。

box groups create --bulk-file-path ./group.csv

この、CSVの作り方がちょっと特殊だったので、忘れないように書いておく。


## CSVの書き方

[Box CLIのヘルプ記事](https://developer.box.com/docs/box-cli)を読んでも、Note: CSV files are also supported for bulk actions, but require special CSV templates.

としか書いてなくて、役にたたない。

[ソース](https://github.com/box/boxcli/blob/30cc7351c9ffedd13df91cff2c03c864f6f738e3/BoxCLI/CommandUtilities/CsvModels/BoxGroupRequestMap.cs)を探ってみた。

ふむふむ。
Name（グループ名), InvitabilityLevel(グループを招待出来る人), MemberViewabilityLevel(グループのメンバーを見れる人)をヘッダーに入れる必要があるそうな。
後ろふたつの有効な値は、admins_only, admins_and_members, all_managed_users

こんな感じのCSVが必要になります。

|Name |InvitabilityLevel |MemberViewabilityLevel |
|:---|:---|:---|
|社長室|admins_only       |all_managed_users      |
|営業部|admins_and_members|admins_and_members     |


# 備忘録終わり

