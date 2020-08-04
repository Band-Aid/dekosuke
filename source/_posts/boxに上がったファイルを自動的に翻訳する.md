---
title: boxに上がったファイルを自動的に翻訳する
date: 2018-06-19 00:11:31
tags:
- box
- documenttranslator
---

英語で共有されたドキュメント。何書いてあるかわからなくて辛いですよね。

誰かが日本語に訳してくれたらどれだけ人生楽になることか。。。

こんなこと考えたことはありませんか？それ、できるんです。
そう、Boxならね。

{% raw %}
  <iframe src="https://cloud.app.box.com/embed/s/ylcid3ibyws8s3qwx9vfwirzcb7dm6lt" width="600" height="400" frameborder="0" allowfullscreen webkitallowfullscreen msallowfullscreen></iframe> 
{% endraw %}

って、正直Box関係ないんですけど、MSFTのDocument Translatorを使ってBoxに上がったドキュメントを自動的に日本語訳版作ってくれる仕組みを30分くらいで作ってみた。

(今年１月の[box hackathon](https://dbox-blog.firebaseapp.com/2018/06/09/Box%20Hackathon%202018/)で提出したもう一個ネタエントリーです。)

[Document Translotor](https://github.com/MicrosoftTranslator/DocumentTranslator/blob/master/README.md)とは、Microsoftが作ったTranslation APIを使ったOfficeドキュメントを指定の言語に翻訳してくれるプログラムです

このDocument Translatorですが、コマンドラインでも実行できることに目をつけて、Box Syncのフォルダを監視対象として、新しいファイルができたらDocument Translatorに翻訳させるという仕組みを思いついたわけです。

_注意：ネタアプリなので超絶雑仕様です。_

# 準備するもの

1. Windows Machine
1. [Document Translator](https://github.com/MicrosoftTranslator/DocumentTranslator/releases)
1. [Microsoft Translation Text API Key](https://docs.microsoft.com/en-us/azure/cognitive-services/translator/)
2. [Box account](https://box.com)
3. [Box Sync (Drive)](https://www.box.com/resources/downloads)

## Setup

1. Document TranslatorをGithubからダウンロード
2. 起動するとAPI Keyを入れる欄があるので、そこに取得したTranslation API Keyをいれます
3. Box Sync もしくは、Box Driveを入れる

以下のようなPSファイルを用意して、実行します。
すると、Box Syncのフォルダに追加されたファイルは自動的に翻訳してBoxに戻してくれるちゅうわけです。

ね？簡単でしょ。

``` powershell
cd "C:\Program Files (x86)\Microsoft Document Translator"
#box上の監視対象とするフォルダに移動します
$folder = 'C:\Users\Administrator\Box Sync\Translation Request'
$filter = '*.*' #<---監視対象の拡張子。雑に全部

#翻訳終わったものを移動するフォルダ
$destination = 'C:\Users\Administrator\Box Sync\Translation Request\Translated'
$fsw = New-Object IO.FileSystemWatcher $folder, $filter -Property @{
 IncludeSubdirectories = $false # <-- サブフォルダ監視する？
 NotifyFilter = [IO.NotifyFilters]'FileName, LastWrite'
}
#フォルダへのアイテム新規作成イベントを登録
$onCreated = Register-ObjectEvent $fsw Created -SourceIdentifier FileCreated -Action {
 $path = $Event.SourceEventArgs.FullPath
 $name = $Event.SourceEventArgs.Name
 $changeType = $Event.SourceEventArgs.ChangeType
 $timeStamp = $Event.TimeGenerated
 
 #doc translatorしたファイルは末尾に-jaとか翻訳した言語名が入る。この文字列がファイル名にが入ってなかったら翻訳する。
 #このあたりもっといい条件づけするといいと思う↓。
 if ($path -like '*en*'){ 
 Move-Item $path -Destination $destination -Force -Verbose
 # Force will overwrite files with same name
 
 }
 #}
 else{./DocumentTranslatorCmd.exe translatedocuments /documents:$path /from:en /to:ja}

# Move-Item $path -Destination $destination -Force -Verbose
}
```

### 補足

監視イベントの登録解除
``` powershell
Unregister-Event -SourceIdentifier FileCreated
```