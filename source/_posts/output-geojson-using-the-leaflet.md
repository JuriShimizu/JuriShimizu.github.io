title: GeoJSONのデータをleafletを使って出力
date: 2015-03-22 14:08:20
tags:
- leaflet
- osm
- geojson
---

最近ずっとPostGISをいじっていろいろやっていたわけですが、
そろそろ地図を動かしたりしてみたい！！
ということで[leaflet](http://leafletjs.com/)に挑戦してみます。

<!-- more -->

東京の観光名所のデータを地図上に表示することを目標にします。

地図上に大量のマーカーを置く時は、[GeoJSON](http://ja.wikipedia.org/wiki/GeoJSON)を利用するとらしいので、
観光地のデータはGeoJSON形式にします。

そのためにやらなくてはいけないことは、、、

1. leafletで地図を表示
1. 地図にマーカーを表示
1. GeoJSONの観光地データを準備
1. 地図にGeoJSONのデータを表示

という感じになります。

ひとつひとつやっていきます。

## leafletで地図を表示

leafletとは地図のためのJavaScriptライブラリ。
OSMなどのオープンデータを簡単に表示することができます。

`<head>`タグ内でCSSとLeafletを読み込みます。

```html
<script src="http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.js"></script>
<link rel="stylesheet" href="http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.css">
```
地図を描画したいところにタグを置きます。
今回は`<body>`タグの中におきました。

```html
<div id="map"></div>
```
CSSで地図の幅、高さを指定
```css
#map{
    width: 100%;
    height: 500px;
}
```
`<script>`タグ内で地図のデフォルト値を設定し、地図を描画します。
今回は東京タワーの場所をデフォルトにしました。

```javascript
//地図のデフォルト値
var map = L.map('map').setView([35.65863174　, 139.74542422], 14);

//OSMレイヤー追加
L.tileLayer(
	'http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
	,{
	    attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a>',
	    maxZoom: 18
	}
).addTo(map);
```
地図を表示するのはたったこれだけで完了！

## 地図にマーカーを表示

東京タワーの場所にマーカーを表示します。
```javascript
var marker = L.marker([35.65863174　, 139.74542422])
    .bindPopup("<h3>東京タワー</h3>")
    .addTo(map);
```

こんな感じで表示できました！

![leaflet](leaflet_1.png)

## GeoJSONの観光地データを準備

PostGISにいれたOSMのデータから観光地のデータを抜き出します。

`As_Geojson`を使うと、GeoJSON形式で出力できるようです。
検索条件については[こちら](http://www.jurigis.me/2015/02/28/conditions-in-postgis/)を参照。

```sql
SELECT row_to_json(feature)
FROM (
  select
    'Feature' AS type,
    ST_AsGeoJSON(ST_Transform(trm.way,4326))::json AS geometry,
    row_to_json((
      SELECT p FROM (
        SELECT trm.name
      ) AS p
    )) AS properties
  from
    planet_osm_point trm
    ,planet_osm_polygon tky
  where
    tky.name = '東京都'
    and tky.boundary = 'administrative'
    and trm.tourism is not null
    and trm.name is not null
    and ST_Contains(tky.way,trm.way)
) AS feature
;
```
こうすると、こんな感じの検索結果になります。
```json
 {"type":"Feature","geometry":{"type":"Point","coordinates":[139.729260969094,35.6604319722521]},"properties":{"name":"森美術館"}}
```

## GeoJSONのデータを地図で表示。

`<script>`タグ内でGeoJSONのデータを定義
先ほど取得したデータを書いておきます。

```javascript
var geojsonFeature = [
  {"type":"Feature","geometry":{"type":"Point","coordinates":[139.729260969094,35.6604319722521]},"properties":{"name":"森美術館"}}
]
```
マーカーをセットします

```
L.geoJson(geojsonFeature).addTo(map);
```
マーカーをクリックしたときにポップアップ表示をさせる場合はこんな感じです。

```javascript
L.geoJson(
geojsonFeature,
{
    onEachFeature: function(feature, layer){
        layer.name(feature.properties.popupContent);
    }
}
).addTo(map);
```
## GeoJSONのデータを外だしする

大量のデータをhtmlファイルに書いていくのはあんまりイケてないので、
GeoJSONのデータを外だしします。

jQueryを使うと簡単らしいので、やってみます。

こんな感じで前項で取得したデータをもとにGeoJsonのデータファイルを用意します。
ディレクトリは`./data/map.geojson`とします。

```json
{
  "type": "FeatureCollection",
  "features": [
  {"type":"Feature","geometry":{"type":"Point","coordinates":[139.74542422,35.65863174]},"properties":{"name":"東京タワー (Tokyo Tower)"}}
  ,{"type":"Feature","geometry":{"type":"Point","coordinates":[139.729260969094,35.6604319722521]},"properties":{"name":"森美術館"}}
・・・・
 ]
}
```

`<head>`タグ内でjQueryを読み込みます。

```html
<script src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
```

データを読み込んで、表示します。

```javascript
$.getJSON("./data/map.geojson", function(data) {
    var geojson = L.geoJson(data, {
        onEachFeature: function (feature, layer) {
            layer.bindPopup(feature.properties.name);
        }
    });
    geojson.addTo(map);
});
```

こんな感じで、マーカーがたくさん表示されました！

![leaflet](leaflet_2.png)

あんまり難しいことはやっていないのですが、
これだけでかなり時間がかかってしまいました、、、

自分で地図を動かせるのは面白いので、
これからもちょいちょい勉強していきたいと思います！

### 参考にさせていただきました

[ぬるぬる動く！Web地図クライアント「Leaflet」を使おう！ |  #GUNMAGISGEEK](http://shimz.me/blog/leaflet-js/4142)
[Leaflet - OSMにgeoJSONでマーカーをつけたりできる便利なJavaScriptライブラリ | アンギス](http://unguis.cre8or.jp/web/6341)
[PostGISからGeoJSON出力するときに属性を付与する - Qiita](http://qiita.com/kshigeru/items/8940ecf7f261a6b01ed0)
[External GeoJSON and Leaflet: The Other Way(s)](http://lyzidiamond.com/posts/external-geojson-and-leaflet-the-other-way/)

