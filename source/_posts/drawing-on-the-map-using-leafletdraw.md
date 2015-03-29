title: Leaflet.drawを使って、地図上に図形を描く
date: 2015-03-29 14:04:06
tags:
- leaflet
---

今日は[Leaflet.draw](https://github.com/Leaflet/Leaflet.draw)というプラグインを使って、
leafletの地図上に図形やマーカーを追加する機能を作ってみます。

<!-- more -->

leafletの地図の表示については[こちら](http://www.jurigis.me/2015/03/22/output-geojson-using-the-leaflet/)を参照

まずは、スタイルシートとjsを読み込みます。

```html
<link href='https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-draw/v0.2.2/leaflet.draw.css' rel='stylesheet' />
<script src='https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-draw/v0.2.2/leaflet.draw.js'></script>
```

追加した図形をまとめるためのオブジェクト`FeatureGroup`を宣言し、
図形を追加するためのコントローラを追加します。

```javascpipt
// 図形をまとめるためのオブジェクト
var drawnItems = new L.FeatureGroup().addTo(map);

// 図形を描くためのコントローラ
// 位置のデフォルトは左上ですが、右上に指定を変えています。
var drawControl = new L.Control.Draw({
  edit: {
    featureGroup: drawnItems
  }
  ,position: 'topright'
}).addTo(map);
```
図形が描かれたときに、
その図形をレイヤに追加するよう設定します。

```javascript
map.on('draw:created', function(e) {
  drawnItems.addLayer(e.layer);
});

```

こうすると地図上にコントローラが現れ、
ポリライン、ポリゴン、四角形、丸、マーカーを、追加したり編集したり削除したりできるようになります。

![leaflet.draw](leaflet.draw_1.png)

もちろんいろいろなカスタマイズも可能です。
試しにこんなカスタマイズをしてみました。
- 追加できるのをポリラインとマーカーだけにする
- ポリラインの色と透明度を変える
- マーカーを連続で作成できるようにする

```
var drawControl = new L.Control.Draw({
  draw: {
    polyline: {
      shapeOptions:{
        color: '#C3415D',
        opacity: 0.8
      }
    }
    ,polygon: false
    ,rectangle: false
    ,circle: false
    ,marker: {
      repeatMode: true
    }
  }
  ,edit: {
    featureGroup: drawnItems
  }
  ,position: 'topright'
}).addTo(map);
```

![leaflet.draw](leaflet.draw_2.png)

この技術を生かして、自分で地図上の情報を編集したり、参照したりできるようにしたいな〜
と思いつつ、今日はここまで〜

