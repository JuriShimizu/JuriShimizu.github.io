title: leafletに縮尺を追加 & ズームコントローラをスライダーに変更
date: 2015-03-23 15:04:22
tags: leaflet
---

leafletの地図を見やすく＆操作しやすくするための小ネタです。

<!-- more -->

leafletの地図の表示については[こちら](http://www.jurigis.me/2015/03/22/output-geojson-using-the-leaflet/)を参照

## 縮尺を追加

やっぱり縮尺があったほうが地図は見やすい！
というわけで、縮尺を追加します。

この１行を追加するだけです！

```javascript
L.control.scale().addTo(map);
```
左下に、メートル単位、フィート単位の縮尺が追加されました。

![scalebar](leaflet_1.png)

フィートは日本では一般的ではないので、消しちゃいましょう。
`imperial`の値をfalseにします。

```javascript
L.control.scale({imperial:false}).addTo(map);
```

メートルの縮尺だけになりました！

![scalebar](leaflet_2.png)

他にも地図を重ねたり、切り替えたり、いろんなことが出来るようです。
leafletの[公式HP](http://leafletjs.com/reference.html)を参照してみてください。

## ズームコントローラをスライダーに変更

leafletのズームコントローラはデフォルトでは、`＋` `ー`ボタンになっています。

![zoomControl](leaflet_3.png)

これをスライダーに変更してみます。
これはleafletではサポートしていないので、[mapbox](https://www.mapbox.com/)の機能を使います。

まずは`<head>`タグ内で、ズームスライダーを読み込みます。

```html
<script src='https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-zoomslider/v0.7.0/L.Control.Zoomslider.js'></script>
<link href='https://api.tiles.mapbox.com/mapbox.js/plugins/leaflet-zoomslider/v0.7.0/L.Control.Zoomslider.css' rel='stylesheet' />
```
`zoomsliderControl`の表示を`true`に、
デフォルトの`zoomControl`の表示を`false`にします。

```javascript
var map = L.map('map',{ zoomsliderControl: true,zoomControl: false}).setView([35.65863174　, 139.74542422], 14);
```

こうすると、スライダーが表示されます。

![zoomControl](leaflet_4.png)




