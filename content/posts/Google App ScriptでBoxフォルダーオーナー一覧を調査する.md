---
title: Google App ScriptでBoxフォルダオーナー一覧を調査する
tags:
 - GAS 
 - box
 - google
 - appscript
---

# はじめに
boxを長く使っていると、色んな人に招待されてルートフォルダが若干カオスな状態になりがちです。
整理をしようにもフォルダーオーナーが誰なのか分からず、確認したくても一個一個目視していくのも結構大変
管理者に頼んでリストを出してもらうのありだけど、管理者もヒマじゃないだろうし、出来ることなら自分でリストを出したい。
![](https://cloud.box.com/shared/static/qefb6l7l4bixjnzfwy2e52q5s7afixiw.png)
---


# Google App Scriptを使ってフォルダーオーナーの一覧を出力する

一覧が欲しいだけなら、Box APIを叩いて一覧をjsonで取得は出来るが可読性が低いし、情報量が多すぎる。
もっと手軽にリスト化する方法はないか。

💡ピコーンッ！Google App Script使って欲しい情報だけセルに落とし込めばいいんじゃね？
それと、生成されたレポートをBox上にアップロード出来るようにしたら、二度うまい。

[デモ動画：やってみた。](https://cloud.app.box.com/s/ymq0kok9btd1ttp0iohfsn8iwmrc2kfg)
--- 

## Google App Script

Google App Sciprt通称GAS（ガースー黒光り探偵社じゃないよ）は、平たく言うとOfficeでいうところのマクロにあたる存在。
大きな違いはGapps上で実行されることと、基本構文がJavascriptに近い
参考：[はじめてのGoogle App Scripts](https://qiita.com/syokoyama/items/d071b1df2b66ba70f564)

まずは、自分のフォルダ一覧を取得するスクリプト

```Javascript
function getNames(){
  //box api: folder一覧をリストするエンドポイント
  aUrl = "https://api.box.com/2.0/folders/0/items";
  //box API Access Tokenの入力をプロンプトで求める
  //開発者トークンでも可能
  var atoken = Browser.inputBox('Access token?', 'access token', Browser.Buttons.OK_CANCEL);
  
  //Get requestのヘッダー情報を用意
  headers = {'Authorization' : 'Bearer ' + atoken};
  params = {'headers' : headers};  
  
  //Folder情報を取得
  var response = UrlFetchApp.fetch(aUrl ,params); // get api endpoint
  var dataAll = JSON.parse(response.getContentText()); //
  var data = dataAll.entries; //parse text into json
  
  //Google Sheetの取得
  var ss = SpreadsheetApp.getActiveSpreadsheet(); //get active spreadsheet
  var sheet = ss.getActiveSheet(); //get sheet by name from active spreadsheet
　Logger.log(data); //log data to logger to check
 
  //ヘッダーカラム情報を入力
  var columns=[];
  columns.push("Folder name");
  columns.push("Folder ID");
  columns.push("Folder Owned By");
  sheet.appendRow(columns);
  columns.splice(0, columns.length);

  //Folder情報の取得
  for (i in data){
    if (data[i].type === 'folder'){
    folderOwner(data[i].id, atoken);
    }
  }
}

function folderOwner(folderid,atoken){
  //Folder情報取得するためのエンドポイント
  aUrl = "https://api.box.com/2.0/folders/";
  headers = {'Authorization' : 'Bearer ' + atoken};
  params = {'headers' : headers};  
  var response = UrlFetchApp.fetch(aUrl + folderid ,params); // get api endpoint
  var dataAll = JSON.parse(response.getContentText()); //
  var stats=[]; //create empty array to hold data points
  var ss = SpreadsheetApp.getActiveSpreadsheet(); //get active spreadsheet
  var sheet = ss.getActiveSheet(); //get sheet from active spreadsheet
　Logger.log(data); //log data to logger to check
 
  stats.push(dataAll.name);
  stats.push(dataAll.id);
  stats.push(dataAll.owned_by.login);
  sheet.appendRow(stats)           
  stats.splice(0, stats.length);
}
```

*今回のスクリプトでは、Access Tokenを直接入れるよう求めているが、OAuth認証フローをGoogle App Scriptに入れることも技術テジュには可能。
これを使うと出来るらしい。。。が時間がなく試していない
https://github.com/googlesamples/apps-script-oauth2


次に、出力したGsheetをExcelに変換して、Boxにアップロードするスクリプト

```Javascript
function exportFile(){
  
  //box API Access Tokenの入力をプロンプトで求める
  //開発者トークンでも可能
  var atoken = Browser.inputBox('Access token?', 'access token', Browser.Buttons.OK_CANCEL);
  var ss = SpreadsheetApp.getActive();
  
  //GsheetをExcelに変換するためのエンドポイント
  var url = "https://docs.google.com/feeds/download/spreadsheets/Export?key=" + ss.getId() + "&exportFormat=xlsx";
  
  //Gsheetの認証情報を用意する
  var params = {
  method      : "get",
  headers     : {"Authorization": "Bearer " + ScriptApp.getOAuthToken()},
  //muteHttpExceptions: true
  };
  
  //Excelデータを取得
  var blob = UrlFetchApp.fetch(url, params).getBlob();
  var blobName = blob.setName(ss.getName() + ".xlsx");


  //Boxにファイルをアップロードする準備。Multipart formアップロードにするため色々用意
  //詳しくはこちら→https://developer.box.com/v2.0/reference#upload-a-file
  var boundary = "boxob";
  //アップロードするFolderID直打ちにしていますが、好きにカスタマイズ出来る。
  var attributes = "{\"name\":\"FolderList.xlsx\", \"parent\":{\"id\":\"40985326479\"}}";
  
  var requestBody = Utilities.newBlob(
    "--"+boundary+"\r\n"
    + "Content-Disposition: form-data; name=\"attributes\"\r\n\r\n"
    + attributes+"\r\n"+"--"+boundary+"\r\n"
    + "Content-Disposition: form-data; name=\"file\"; filename=\""+blobName+"\"\r\n"
  + "Content-Type: " + blob.getContentType()+"\r\n\r\n").getBytes()
  .concat(blob.getBytes())
  .concat(Utilities.newBlob("\r\n--"+boundary+"--\r\n").getBytes());
  
  var options = {
    method: "post",
    contentType: "multipart/form-data; boundary="+boundary,
    payload: requestBody,
    muteHttpExceptions: true,
    headers: {'Authorization': 'Bearer ' + atoken}
  };

  //Boxにアップロード
  var request = UrlFetchApp.fetch("https://upload.box.com/api/2.0/files/content", options);

  Logger.log(request.getContentText());
  
}
```

# この連携にあたって参考にした情報

[Google Apps ScriptからHTTP POST](https://qiita.com/n0bisuke/items/a31a99232e50461eb00f)
[External APIs](https://developers.google.com/apps-script/guides/services/external)
