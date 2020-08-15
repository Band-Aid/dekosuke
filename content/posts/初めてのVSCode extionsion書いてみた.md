---
title: QiitaにVSCodeから直接投稿したい
tags: 
- VSCode
- Qiita
---
Qiitaの記事を一度VSCodeで下書きして書いているのですが、いちいち出来上がった記事をQiita開いてコピペして投稿するのがめんどくさい。。。
VSCodeから直接投稿できたら楽だなぁ、とずって思ってた。

誰か便利なExtension書いてくれていないかなと思って探してみたけど、誰も作ってないっぽい。。。
VSCodeとES6の勉強を兼ねて、作ってみた。

## Qiita API
まずは、QiitaのAPIを調べてみました
QiitaのAPIはとてもシンプル。これが、ベースURL https://qiita.com/api/v2/
投稿をするには、/itemsにヘッダーをつけてPOST

アクセストークンは[一旦発行](https://qiita.com/api/v2/docs#%E8%AA%8D%E8%A8%BC%E8%AA%8D%E5%8F%AF)すればずっと使えるみたい。

ここから[発行](https://qiita.com/settings/applications)します。
一度しかアクセストークンは表示されないのでしっかり書き留めておきましょう。
![](https://cloud.box.com/shared/static/syo3chx0kbpc0j7gxmseq9wd2oinhcuh.png) 

## VSCodeのHello World
VSCodeのサンプルコードを参考にプロジェクトを立ち上げる
https://code.visualstudio.com/docs/extensions/example-hello-world


### VSCodeのロジックを書く

**Qiita API用ライブラリを書く**

```javascript

require('dotenv').config();
const Client = require('node-rest-client').Client
const client = new Client()
const key = process.env.KEY;
const accessToken = key
const baseurl = 'https://qiita.com/api/v2/'

class QiitaAPI{

static post(body,title){
    var args = {
        data: 
        {
          "body": body,
          "coediting": false,
          "gist": false,
          "group_url_name": "dev",
          "private": true,
          "tags": [
            {
              "name": "box",
              "versions": [
                "0.0.1"
              ]
            }
          ],
          "title": title,
          "tweet": false
        },
        headers: {"Authorization":"Bearer "+ accessToken, "Content-Type": "application/json" }
    };

client.post(baseurl + 'items',args,function (data, response){
    console.log(data)
    console.log(response.statusCode)
      if(response.statusCode >= '400'){
         vscode.window.showInformationMessage('failed to upload with error ' + response.statusCode)
      }
      if(response.statusCode >= '201'){
         vscode.window.showInformationMessage('hooray! successfully uploaded')
      }
    })
  }
}
module.exports = QiitaAPI;
```

Node-rest-clientの[サンプルコード](https://www.npmjs.com/package/node-rest-client)でreq.onでエラーハンドルしろって書いてあるが、いくらやっても.onを正しいプロパティとして認識させることができなかった。**エラーハンドリングは今後の課題**


### package.json
今回は特定のコマンド(upload)をVSCodeから実行したら記事のアップロードを実行するようにした。

```json:package.json
{
    "name": "go-qiita",
    "displayName": "go Qiita",
    "description": "upload new article to Qiita",
    "version": "0.0.1",
    "publisher": "Daichi",
    "engines": {
        "vscode": "^1.21.0"
    },
    "categories": [
        "Other"
    ],
    "activationEvents": [
        "onCommand:extension.uploadFile"
    ],
    "main": "./extension",
    "contributes": {
        "commands": [
            {
                "command": "extension.uploadFile",
                "title": "upload"
            }
        ]
    },
    "scripts": {
        "postinstall": "node ./node_modules/vscode/bin/install",
        "test": "node ./node_modules/vscode/bin/test"
    },
    "devDependencies": {
        "typescript": "^2.6.1",
        "vscode": "^1.1.6",
        "eslint": "^4.11.0",
        "@types/node": "^7.0.43",
        "@types/mocha": "^2.2.42"
    },
    "dependencies": {
        "node-rest-client": "^3.1.0"
    }
}

```

### extension.js

```javascript
// The module 'vscode' contains the VS Code extensibility API
// Import the module and reference it with the alias vscode in your code below
const vscode = require('vscode');
const QiitaAPI = require('./qiita-api')
const path = require('path')
// this method is called when your extension is activated
// your extension is activated the very first time the command is executed
function activate(context) {

    // Use the console to output diagnostic information (console.log) and errors (console.error)
    // This line of code will only be executed once when your extension is activated
    console.log('Congratulations, your extension "go-qiita" is now active!');

    // The command has been defined in the package.json file
    // Now provide the implementation of the command with  registerCommand
    // The commandId parameter must match the command field in package.json
    let disposable = vscode.commands.registerCommand('extension.uploadFile', function () {
        // The code you place here will be executed every time your command is executed
        let editor = vscode.window.activeTextEditor;
        if (!editor) {
            return; // No open text editor
        }
       
        let firstLine = editor.document.lineAt(0);
        let lastLine = editor.document.lineAt(editor.document.lineCount - 1);
        let textrange = new vscode.Range(0,firstLine.range.start.character,editor.document.lineCount - 1,lastLine.range.end.character);
        let text = editor.document.getText(textrange);
        let title = path.basename(editor.document.fileName);

        QiitaAPI.post(text,uploadtitle)
    });

    context.subscriptions.push(disposable);
}
exports.activate = activate;

// this method is called when your extension is deactivated
function deactivate() {
}
exports.deactivate = deactivate;
```

とりあえず動いた！この記事も無事アップロードされました。

### 今後の改善リスト
- 全体的なクリーンアップ
  - エラーハンドリング
  - Qiita APIライブラリのクリーンアップ 
- 機能追加
 - 記事の公開可否を指定できるようにする
 - タグのアップロード
 - 既存記事を取得して編集する
