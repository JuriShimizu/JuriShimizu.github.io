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

しかし、、、
[Bootstrap](http://getbootstrap.com/)を使っていると、スライダーの線が消えて、こんなマヌケな姿になってしまします、、、

![zoomControl](leaflet_5.png)

しかもよく見ると、ツマミも小さくなって、つまみづらくなってる、、、

こんなときはcssにこの記述を追加すると、元通りになります。

```css
.leaflet-control-zoomslider-body {
    box-sizing: content-box;
}

.leaflet-control-zoomslider-knob {
    box-sizing: content-box;
}
```

## おまけ：なんでスライダーの線が消えてしまったのか？？？

なぜBootstrapを使うとスライダーの線が消えてしまったのでしょうか？

Bootstrapのcssにはこんな定義があります。

```css
* {
    box-sizing: border-box;
}
```

`box-sizing`はボックスサイズの算出方法を指定します。

`border-box`だと線の太さを幅・高さに含める
`content-box`だと線の太さを幅・高さに含めない

デフォルトは`content-box`です。

スライダーのツマミを動かす部分は、`L.Control.Zoomslider.css`でこのように定義されています。

```css
.leaflet-control-zoomslider-body {
	        width: 2px;
	        border: solid #fff;
	        border-width: 0px 9px 0px 9px;
	        background-color: black;
	        margin: 0 auto;
	}
```

背景色（background-color）が黒、横幅が２ピクセルになっています。
線のように見えていたのは、背景色だったのですね。

`box-sizing: content-box`が指定されている状態だと、
このボックスの幅は、` 2px + 9px（ボーダーの太さ）* 2 = 20 ` となり、
背景色（黒）がみえる部分は 2px になります。

`box-sizing: border-box`が指定されている状態だと、
このボックスの幅は、`2px` となり、
幅がボーダーラインよりも小さくなってしまって、背景色（黒）が見えなくなってしまうのです。

なので、　こんな感じで`.leaflet-control-zoomslider-body`　の
`box-sizing`の定義を書き換えることで、スライダーの線が見えるようになるのです。

```css
.leaflet-control-zoomslider-body {
    box-sizing: content-box;
}
```

ツマミが小さくなってしまったのも、同じ理由になります。

GISとは関係のない話になりましたが、
いろんなcssを読み込むときは注意が必要ですね！

という話でした！



