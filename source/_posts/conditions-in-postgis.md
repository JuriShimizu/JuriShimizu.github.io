title: PostGISでGeoな検索条件
date: 2015-02-28 16:09:17
tags:
- postgis
- osm
---

すっかり勉強をサボってしまいましたが、
前回に引き続き、OSMのデータを使ってPostGISのGeoなSQLを学んでいきます！

今回はGeoな検索条件です。

<!-- more -->

#### ST_Within(geometry A , geometry B)

ジオメトリAが完全にジオメトリBの中に含まれているときにTRUEを返します。

台東区内にあるファストフード店をselectしてみます

```sql
japan=# select
japan-#     fst.name
japan-# from
japan-#     planet_osm_point fst
japan-#     ,planet_osm_polygon tit
japan-# where
japan-#     tit.name like '台東%'
japan-#     and tit.boundary = 'administrative'
japan-#     and tit.admin_level = '7'
japan-#     and fst.amenity = 'fast_food'
japan-#     and ST_Within(fst.way,tit.way)
japan-# ;
            name
----------------------------
 麺昇七辻
 かめや
 東京チカラ飯
 フレッシュネスバーガー
 吉野家
 奥州麺処 秘伝
 川中島うどん店
 日高屋
 富士そば
 サブウェイ
 ケンタッキーフライドチキン
 マクドナルド
 回転寿司 大江戸
 味噌麺
 マクドナルド
 味噌屋 せいべえ
 吉野家
 かつや
 富士そば
 Freshness Burger
 楽釜製麺所
 まぐろ人
 寿司食べ放題
 藪そば
 大江戸 回転寿司
 東尋坊
 てん屋

 元祖寿司
 牛の力

 モスバーガー

 Yoshinoya
 すき家
 McDonalds
 たい焼き たいち
 麺八
 松屋
 日本海
 Sukiya
 デリカ ぱくぱく
 モスバーガー
 ら麺亭 (Ramen-tei)
 松屋
 ケンタッキーフライドチキン
 銀だこ (Gindaco)
 CoCo壱番屋
(48 rows)
```

ファストフード？？？みたいなお店もありますが、ちゃんとselectできました。

#### ST_Contains(geometry A, geometry B )

ST_Withinの真逆です。

ジオメトリBが完全にジオメトリAの中に含まれているときにTRUEを返します。

ST_Withinの代わりにST_Containsを使って、引数をひっくり返すと同じ結果になります。

```sql
japan=# select
japan-#     fst.name
japan-# from
japan-#     planet_osm_point fst
japan-#     ,planet_osm_polygon tit
japan-# where
japan-#     tit.name like '台東%'
japan-#     and tit.boundary = 'administrative'
japan-#     and tit.admin_level = '7'
japan-#     and fst.amenity = 'fast_food'
japan-#     and ST_Contains(tit.way,fst.way)
japan-# ;
            name
----------------------------
 麺昇七辻
 かめや
 東京チカラ飯
・・・・・
(48 rows)
```

#### ST_Equals(geometry A , geometry B)

AとBがまったく同じジオメトリのときにTRUEを返します。

`ST_Equals(A,B)`は、
`ST_Within(A,B) and ST_Contains(A,B)`と同じ意味になります。

```sql
japan=# select
japan-#     fst.osm_id
japan-#     ,fst.name
japan-# from
japan-#     planet_osm_polygon fst
japan-#     ,(select way from planet_osm_polygon where name like '東京%' and boundary = 'administrative') tky
japan-# where
japan-#     ST_Equals(fst.way,tky.way)
japan-#     and name like '東京%'
japan-# ;
  osm_id  |  name
----------+--------
 -1543125 | 東京都
 -1543125 | 東京都
 -1543125 | 東京都
 -1543125 | 東京都
 -1543125 | 東京都
 -1543125 | 東京都
 -4479121 | 東京都
 -1543125 | 東京都
(8 rows)
```

なぜか、まったく同じ東京がたくさん。。。
いろんな人が登録しちゃったってことですかね。。。

#### ST_DWithin

ジオメトリが，指定したジオメトリから指定した距離内にある場合に，TRUEを返します。
メートル単位で距離を指定するためにはgeography型にする必要があるようです。

東京駅の半径１km以内にあるファストフードを検索してみます

```sql
japan=# select
japan-#     fst.name
japan-# from
japan-#     planet_osm_point fst
japan-#     ,(select way from planet_osm_point  where name = '東京') tky
japan-# where
japan-#     ST_DWithin(geography(ST_Transform( fst.way,4326 )),geography(ST_Transform( tky.way,4326 )),1000)
japan-#     and amenity = 'fast_food'
japan-# ;
        name
--------------------
 ロッテリア
 よもだそば
 喜多方ラーメン坂内
 サブウェイ
 吉野家
 カレーショップC&C
 吉野家
 吉野家
 サブウェイ
 マクドナルド
 松屋
 富士そば
 小諸そば
 小諸そば
 なか卯
(15 rows)
```
１５件ほどひっかかりました！

型の変換をしないとこんな感じで検索結果が変わってしまいます。
```sql
japan=# select
japan-#     fst.name
japan-# from
japan-#     planet_osm_point fst
japan-#     ,(select way from planet_osm_point  where name = '東京') tky
japan-# where
japan-#     ST_DWithin(fst.way,tky.way,1000)
japan-#     and amenity = 'fast_food'
japan-# ;
       name
-------------------
 吉野家
 カレーショップC&C
 吉野家
 サブウェイ
 マクドナルド
 松屋
 富士そば
 小諸そば
 小諸そば
 なか卯
(10 rows)
```

ST_DWithinの引数は、ラインやポリゴンでもOKです。

#### ST_Touches

ジオメトリが共通のポイントを少なくとも一つ持ち，内部でインタセクトしない場合に，TRUEを返します。
つまり、接しているか？ってことですね

台東区に接している区を調べてみましょう！

```sql
japan=# select
japan-#     cty.name
japan-#     ,cty.admin_level
japan-# from
japan-#     planet_osm_polygon cty
japan-#     ,planet_osm_polygon tit
japan-# where
japan-#     tit.name like '台東%'
japan-#     and tit.boundary = 'administrative'
japan-#     and tit.admin_level = '7'
japan-#     and cty.boundary = 'administrative'
japan-#     and cty.admin_level = '7'
japan-#     and ST_Touches(tit.way,cty.way)
japan-# ;
      name       | admin_level
-----------------+-------------
 中央区          | 7
 千代田区        | 7
 墨田区 (Sumida) | 7
 荒川区          | 7
(4 rows)
```

あれ？文京区とかも接しているはずなんですが、、、

検索してみたところ、文京区のデータがそもそもありませんでした。
OSMにデータ登録されてないのか、うまくデータインポートできていないのかはナゾです。。。

他にもクロスしてる〜とか、ジオメトリの関係を表す検索条件はいっぱいあるようです！

SQLはずっと使っていますが、Geoな条件はちょっと勝手が違って戸惑いました。。。
SQLがあんまりキレーじゃないのは許してください。。。

今日はここまで！

