title: Bootstrapを使うとleafletのズームスライダーがキレイに表示されない
date: 2015-03-29 12:55:36
tags:
- leaflet
- bootstrap
---

[前回](http://www.jurigis.me/2015/03/23/zoomslider-in-leaflet/)、leafletでズームスライダーを表示する方法をご紹介しましたが、
実は、[Bootstrap](http://getbootstrap.com/)を使っていると、ズームスライダーがうまく表示されなくなります。
今回はその解消方法について書きます。

<!-- more -->

leafletの地図の表示については[こちら](http://www.jurigis.me/2015/03/22/output-geojson-using-the-leaflet/)を参照
leafletの地図にズームスライダーを表示する方法については[こちら](http://www.jurigis.me/2015/03/23/zoomslider-in-leaflet/)を参照

## 事象

まず、Bootstrapを使っていない場合のスライダーはこんな感じ

![zoomslider](leaflet_1.png)

Bootstrapを使っていると、こんなマヌケな姿に、、、

![zoomslider](leaflet_2.png)

真ん中の線が消えて、
ツマミも小さくなってつまみづらくなっています。。。

## 対策

cssにこの記述を追加すると、元通りになります。

```css
.leaflet-control-zoomslider-body {
    box-sizing: content-box;
}

.leaflet-control-zoomslider-knob {
    box-sizing: content-box;
}
```

## 原因

なぜBootstrapを使うとスライダーの線が消えてしまったのでしょうか？

Bootstrapのcssにはこんな定義があります。

```css
* {
    box-sizing: border-box;
}
```

`box-sizing`はボックスサイズの算出方法を指定します。

`border-box`だと線（border）と余白（padding）を幅・高さに含める
`content-box`だと線（border）と余白（padding）を幅・高さに含めない

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
