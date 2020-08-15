---
title: Microsoft Azure Logic Appsを使ってSharepoint + Box連携
tags: 
 - box 
 - AzureLogicApps
 - SharePoint
---
# Microsoft Azure Logic Appsを使ってSharepoint + Box連携

[Microsoft Azure Logic App](https://azure.microsoft.com/ja-jp/services/logic-apps/)とは、IFTTTなどと同様の[iPaaS](https://www.accenture.com/jp-ja/blogs/cloud-blog-japan-ipaas-cloud)アプリケーション連携ツールになります。
予め様々なSaaSアプリケーションとの連携[モデュール](https://docs.microsoft.com/ja-jp/azure/connectors/apis-list)が用意されており、ドラッグ＆ドロップによる簡単操作でアプリ間の連携を容易に自動化することが可能となっています。また、複雑なロジックは開発者自身でプログラミングすることも可能です。

Azure Logic App上には、すでにSharepointとBox用のモデュールが用意されています。
今回は、Sharepointサイトにアップロードされたファイルを検知し、box上に同じものをアップロードするロジックを作ってみます。

[デモ動画](https://cloud.box.com/s/ix5ygev7tyl13ly9jtbejutmcquvgi9b)

# 設定
![](https://cloud.box.com/shared/static/hi4kax53mapjev9d5t952y1e0uoiszlq.png)

## Box上のフォルダに何もない状態
![](https://cloud.box.com/shared/static/b5ftveffvcf6zkmwfbi2nskaixcq2w6d.png)

## Sharepointにファイルをアップロードします
![](https://cloud.box.com/shared/static/crv7ufeaff7iscue9fliw43vrndrd4uz.png)

![](https://cloud.box.com/shared/static/bji2c24yxwjg97qx69q7mkz864vcp1e2.png)

## Azure Logic Appの実行ログを見てみます。

![](https://cloud.box.com/shared/static/ftwjjdkf07lv5sb22kxwzd4gctksh31z.png)

## Box上のフォルダに同じものがアップロードされている

![](https://cloud.box.com/shared/static/117sjdcgm2n9ztnhh4afskczodj4ge9y.png)


## まとめ

SharepointのファイルもすべてBoxへ保存できます！
新規ファイルの検知だけではなく、変更検知も出来るのでsharepoint上のファイルを全てboxに同期することが出来ます。
どうしてもsharepointから脱却できないWFもこれで安心。


