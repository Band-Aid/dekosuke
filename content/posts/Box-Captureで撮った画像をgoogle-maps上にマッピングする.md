---
title: Box Captureで撮った画像をgoogle maps上にマッピングする
date: 2018-07-18 22:10:02
tags:
---

みなさん、Box Capture使っていますか？
写真、動画、音声の記録に特化したアプリで、撮ったデータを直接Boxにアップロードできるiosアプリです。
https://itunes.apple.com/jp/app/box-capture/id1040325411?mt=8

Box Captureには、撮った写真の位置情報をBoxのメタデータとして保存する機能も備わっています。
Box Capture > 設定 > 場所メタデータを保存 On

<img src='https://cloud.box.com/shared/static/q5qvf7pcwy29phefgpniznqklvrr6gx7.png'  hieght='100' width='250'>

この設定をOnにして写真を撮ると、Boxメタデータに値が保存されます。

<img src='https://cloud.box.com/shared/static/lu40vrcn03fkbbk6fd3ay5dose3e5ktn.png' hieght='200' width='500'>

Box Captureで撮った写真をGoogle Mapsにマッピングするアプリを作ってみました。

**実装サンプル：** https://xmlhoster.azurewebsites.net/demomap.html

<img src='https://cloud.box.com/shared/static/rwvrpmz9rgtrdwv94hgarrzrchekdauo.png' height='400' width='500'>

# 準備

## Google maps API

Google mapsにカスタムデータを追加する方法はいくつか用意されていますが、今回はJavscript APIとXMLファイルを使った方法でマッピングします。

Google Map上にデータをマッピングするには、Maps API Keyを準備する必要があります。

こちらのサイトからAPI Keyを取得します。https://cloud.google.com/maps-platform/?hl=ja

取得したAPI Keyは、ライブラリ参照先に入れます。

```javascript
 </script>
  <script async defer src="https://maps.googleapis.com/maps/api/js?key=xxxxxxxxxx&callback=initMap">
  </script>
</body>
```

## Box側の準備

Google mapに配置するためのXMLファイルを用意します。

mapにマーカーを配置するのに必要最低限の情報は、LatitudeとLongtitudeの値です。

でも、それだけだと面白くないので、Map上のマーカーをクリックしたら写真が見れるようにしたいと思います。

最終的なXMLはこんな感じになるよう整形します。

```xml
<markers>
  <marker id='304925721161' lat='35.61233666666666' lng='139.72142' url='https://cloud.box.com/shared/static/jnz4ee648ww7xubz9dx6kesisvnbcbs.jpg'/>
  <marker id='304947099780' lat='n' lng='u' url='https://cloud.box.com/shared/static/xwdd89t7zldgz7cya83pt91rtm0s7nv.jpg'/>
  <marker id='304923042336' lat='35.61233666666666' lng='139.7207333333333' url='https://cloud.box.com/shared/static/n5q5e4qe4b8u8wuk2n01awi42ni785.jpg'/>
...
```

では、Rubyを使ってXMLファイルの準備をします。([Box Ruby SDK](https://github.com/cburnette/boxr)を使用しています。)

### Boxファイルのメタデータを取得

Box CaptureのメタデータはGlobalスコープで、boxCaptureV1というテンプレート名の中に保存されています。
また、Lat,Longの値は一行で保存されているので、不要なデータの文字列処理をしてやる必要があります。(例：35.67489666666667 N, 139.7625716666667 E)

```ruby
def getLocation(id,client)
  begin
   client.metadata(id,template: 'boxCaptureV1')
   location = client.metadata(id,template: 'boxCaptureV1').location.gsub(/\sE/, '').gsub(/\sN/, '').gsub(/\s/, '').split(',')
   return location
  rescue Boxr::BoxrError => e
   return 'null' 
  end
end

```

### makerクリック時に表示する画像の直リンクを取得します。
```ruby
def fileURL(id,client)
  return client.create_shared_link_for_file(id,access: :open).shared_link.download_url
end
```

### XMLの準備

```
Q. なんでわざわざXMLをいちから作るの？

A. データベース立ち上げるのめんどくさかった。量が増えるようならDBで管理したほうがいいんだろうなぁと思ったり。
リアルタイム性を担保するならWebhook連携とか考える必要ありそう。
```

```ruby
def addxml (items,client)
  doc = REXML::Document.new
  #ノードの追加  
  root = REXML::Element.new('markers')
  doc.add_element(root)
  items.each{|i| 
    id = i.id
    if i.type == 'file' then
    location = getLocation(id,client)
    url = fileURL(id,client)
    # ルートノードの下に子ノードを追加
    i.id = REXML::Element.new('marker')
    i.id.add_attribute('id',id)
    i.id.add_attribute('lat', location[0])
    i.id.add_attribute('lng', location[1])
    i.id.add_attribute('url', url)
    root.add_element(i.id)
    end
  }
  File.open('output.xml', 'w+') do |file|
    doc.write(file, indent=2)
  end
  puts 'finished xml output'
end
```

最後に上記メソッドを呼び出します。

```ruby
client = Boxr::Client.new(atoken)
#Box CaptureがファイルをアップロードしたフォルダIDを指定
items = client.folder_items(xxxxxxxxx)
addxml(items,client)
```

## Google Mapに配置する

これで、XMLの準備はできました。

つぎはXMLのデータをMapに配置するためのHTMLページを用意します。

今回はこのサンプルをベースに作っていきます。

https://developers.google.com/maps/documentation/javascript/mysql-to-maps?hl=ja

HTML tagの部分はサンプルを拝借するとして、今回作ったXMLをmapにマッピングするロジックは以下：

**初期化**

```javascript
   function initMap() {
      var map = new google.maps.Map(document.getElementById('map'), {
        center: new google.maps.LatLng(35.3606646, 139.0288265),
        zoom: 6
      });
      //マーカーをクリックしたときに表示するWindowも初期化しておきます。
      var infoWindow = new google.maps.InfoWindow;
```

**XMLのマッピング**

```javascript
//先程作成したXMLをhttp(s)経由で取得する
      downloadUrl('http://localhost/xml', function (data) {
        var xml = data.responseXML;
        var markers = xml.documentElement.getElementsByTagName('marker');
        Array.prototype.forEach.call(markers, function (markerElem) {
          var id = markerElem.getAttribute('id');
          var point = new google.maps.LatLng(
            parseFloat(markerElem.getAttribute('lat')),
            parseFloat(markerElem.getAttribute('lng')));
          var linkurl = markerElem.getAttribute('url');
```

**infowindow**

```javascript
var contentString = '<div id="content">' +
            '<div id="siteNotice">' +
            '</div>' +
            '<a href=https://boxdemobox.app.box.com/file/' + id + ' target="_blank">Boxでファイルを開く</a>' +
            '<h1 id="firstHeading" class="firstHeading">' + markerElem.getAttribute('lat') + ',' + markerElem.getAttribute('lng') + '</h1>' +
            '</br>' +
            '</br>' +
            '<div id="bodyContent">' +
            //linkurlに画像の直リンク情報が入る
            '<p>' + '<img src=' + linkurl + 'height="150" width="400" onclick="Rotate()" class="normal" id="image">' +
            '</br>' +
            '</div>' +
            '</div>'

          var infowindow = new google.maps.InfoWindow({
            content: contentString
          });
```

**マーカーのクリックイベント**

```javascript
          var marker = new google.maps.Marker({
            map: map,
            //icon: icon,
            animation: google.maps.Animation.DROP,
            position: point
          });
          marker.addListener('click', function () {
            //infoWindow.setContent(infowindow);
            infowindow.open(map, marker);
          });
```

あとは、xmlとhtmlファイルを適当なサーバにホストしてできあがり。

ハザードマップとか現地調査マップとして使えるんじゃないでしょうか。

応用編としてBox elements UIとかComment APIをマーカー内に組み込むことで地図上から直接コメントやファイルのやりとりが実現できそうです。

GISソリューションがGoogle MapとBoxだけで作れてしまう！すごい！



**追記：**
読み込んだ画像が何故か横になってしまう現象が発生。よくわからんので、画像をクリックして90度回転するロジック追加：

```css
    .rotate {
      -ms-transform: rotate(90deg);
      -webkit-transform: rotate(90deg);
      transform: rotate(90deg);
    }

    .normal {
      -ms-transform: rotate(0deg);
      -webkit-transform: rotate(0deg);
      transform: rotate(0deg);
    }
```

```javascript
    function Rotate() {
      var element = document.getElementById('image');

      if (element.className === "normal") {
        element.className = "rotate";
      }
      else if (element.className === "rotate") {
        element.className = 'normal';
      }
    }
```
