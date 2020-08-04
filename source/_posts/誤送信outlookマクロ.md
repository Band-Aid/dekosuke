---
title: ご送信対策outlookマクロ
date: 2018-06-08 17:21:51
tags:  
 - outlook 
 - box 
 - マクロ
---
# メール送る前に送信者チェックする
日本人はどうやらメールご送信恐怖症みたいです。
外部にメールを送る前に上司が承認しないとメールが送信されない。そんなことをやっている企業があるという話を聞きました。。。（驚愕）
まぁ、そういう業界ではそういうものなんでしょうか。内情知らないのでとやかく言いません。

でも、上司がチェックしなくても、みんなそれぞれが誤送信対策できるんじゃないんですかね。
昔自分がOutlookを使用してた頃に活用してたマクロを紹介したいと思います。

自分のドメイン(@hogehoge.com)以外のアドレスが宛先に含まれていると、ほんとに大丈夫？ってメッセージを出すマクロです。

## 使い方
最近のOffice製品はデフォルトでマクロが無効化されているので、有効化する必要があります。

1. Outlook > File > options > セキュリティセンター > macro > 全てのマクロに対して警告を表示する
1. Outlook > File > options > ユーザのリボン設定 > 開発タブを表示
1. 開発タブ > Visual Basic > this outlook sessionを開く > 以下スクリプトをコピー＆ペースト > outlook再起動

 
```vb
Private Sub Application_ItemSend(ByVal Item As Object, Cancel As Boolean)
    Dim recipents As Outlook.Recipients
    Dim recipent As Outlook.Recipient
    Dim pa As Outlook.PropertyAccessor
    Dim prompt As String
    Dim strMsg As String
 
    Const PR_SMTP_ADDRESS As String = "http://schemas.microsoft.com/mapi/proptag/0x39FE001E"
 
    Set recipents = Item.Recipients
    For Each recipent In recipents
        Set pa = recipent.PropertyAccessor
        If InStr(LCase(pa.GetProperty(PR_SMTP_ADDRESS)), "@hogehoge.com <= ここ自分のアドレスと置き換えてください") = 0 Then
            strMsg = strMsg & "   " & pa.GetProperty(PR_SMTP_ADDRESS) & vbNewLine
        End If
   Next
 
    If strMsg <> "" Then
        prompt = "宛先に社外のユーザが含まれています。to:" & vbNewLine & strMsg & "本当に送りますか?"
        If MsgBox(prompt, vbYesNo + vbExclamation + vbMsgBoxSetForeground, "Check Address") = vbNo Then
            Cancel = True
        End If
    End If
End Sub
```

自分的には宛先を確認するようになるので、結構有効な対策でした。

# 結論(超強引)：Box使いましょう
機密情報はメールには書かず、[Box Noteに書いておくとか](https://blog.animereview.jp/boxnote/)、ファイルはBoxにアップロードし、共有リンクのアクセス制御をしっかりと行いましょう。
万が一誤送信したとしてもBoxのフォルダにアクセス権がないと読めないし、情報流出も防げます。

