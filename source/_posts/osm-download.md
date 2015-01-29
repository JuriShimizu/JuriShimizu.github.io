title: OSMのデータをPostGISのDBに入れてみる
date: 2015-01-12 17:35:12
tags: 
- vagrant
- centos
- virtualized_environment
- postgis
- osm
---

[前回](http://jurishimizu.github.io/2015/01/09/postgis-install-2/)、Geoなデータベースを作ったので、
今回はそこにGeoなデータを入れてみたいと思います

<!-- more -->

[OSM（オープンストリートマップ）](http://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3%E3%82%B9%E3%83%88%E3%83%AA%E3%83%BC%E3%83%88%E3%83%9E%E3%83%83%E3%83%97)のデータをいれてみます。
OSMは誰でも使用できて、だれでも編集できる地図なのです。
もちろんタダです。
そしてこのOSMを編集する人をマッパーと呼ぶらしいです。地図好きってけっこう多いんですね

他にも[国土数値情報](http://nlftp.mlit.go.jp)とか[自然環境情報GIS](http://www.vegetation.biodic.go.jp/)とか、
タダでダウンロードできるGISデータはたくさんあるようです。
それを組み合わせれば、いろいろ面白そうですね

## osm2pgsqlインストール

OSMのデータを扱うために、[osm2pgsql](http://wiki.openstreetmap.org/wiki/Osm2pgsql)を入れます。
説明が英語だ、、、
OSMのデータをPostGISのDBにいれるためのツールってことでいいかと思います。。。

osm2pgsqlを入れるためは、他にもいろいろインストールが必要なのですが、
その辺もあんまり理解できていないので、
トライ&エラーでやっていきたいと思います。

```sh
$ git clone https://github.com/openstreetmap/osm2pgsql
$ cd osm2pgsql
$ ./autogen.sh
./autogen.sh: line 2: autoreconf: command not found
```

案の定エラーになりました、、、
libtoolが必要らしいので入れます

```sh
$ yum install libtool
$ ./autogen.sh
```
これでautogen.shは成功しました
```sh
$ ./configure --with-protobuf-c
checking for xml2-config... no
configure: error:  does not exist or it is not an exectuable file
```
またエラーだ、、、
libxml12を入れます
```sh
$ yum install libxml2 libxml2-devel
$ ./configure --with-protobuf-c
checking for bzip2 compression library... no
configure: error: required library not found
```
別のエラーが、、、
bzip2をいれます
```sh
$ yum install bzip2 bzip2-devel
$ ./configure --with-protobuf-c
configure: error: geos-config does not exist or it is not an exectuable file
```
今度はgeosです
```sh
$ yum install geos geos-devel
$ ./configure --with-protobuf-c
checking for proj projection library... no
configure: error: required library not found
```
続いてproj、、、、
```sh
$ yum install proj proj-devel
$ ./configure --with-protobuf-c
checking for protobuf-c usability... no
configure: WARNING: protobuf-c support requested but headers or library not found. Specify valid prefix of protobuf-c using --with-protobuf-c=[DIR] or provide include directory and linker flags using --with-protobuf-c-inc and --with-protobuf-c-lib
checking for pg_config... no
configure: error:  does not exist or it is not an exectuable file
```
protobuf-cがない、、、
```sh
$ yum install protobuf-c protobuf-c-devel
$ ./configure --with-protobuf-c
checking for pg_config... no
configure: error:  does not exist or it is not an exectuable file
```
pg_configがみつからないようですが、
これは環境変数を設定すればOKなようです
```sh
$ export PATH=/usr/pgsql-9.3/bin:$PATH
$ ./configure --with-protobuf-c
checking if more special flags are required for pthreads... no
checking for boostlib >= 1.48... configure: We could not detect the boost libraries (version 1.48 or higher). If you have a staged boost library (still not installed) please specify $BOOST_ROOT in your environment and do not give a PATH to --with-boost option.  If you are sure you have boost installed, then check your version number looking in <boost/version.hpp>. See http://randspringer.de/boost for more documentation.
configure: error: cannot find Boost libraries, which are are required for building osm2pgsql. Please install libboost-dev.
```
Boostが必要なようです
これは手でいれないといけないみたい、、、

```sh
$ wget http://downloads.sourceforge.net/project/boost/boost/1.53.0/boost_1_53_0.tar.gz
$ tar -xvzf boost_1_53_0.tar.gz
$ cd boost_1_53_0
$ ./bootstrap.sh --prefix=/usr/local
$ ./b2
$ ./b2 install
$ /sbin/ldconfig
```
```sh
$ ./configure --with-protobuf-c
```
やっと成功しました。あとはmakeすればOK
```
$ make
$ make install
```

やっとできました。。。。
長かった、、、

## OSMのデータをいれる

日本の最新の地図情報を落としてきて、DBに入れてみます

```
$ wget http://download.geofabrik.de/asia/japan-latest.osm.pbf
$ osm2pgsql -c -d japan -S /usr/local/share/osm2pgsql/default.style --host localhost japan-latest.osm.pbf
osm2pgsql: error while loading shared libraries: libboost_filesystem.so.1.53.0: cannot open shared object file: No such file or directory
```
またエラーが、、、
libboost_filesystem.so* の場所を探して、環境変数に追加
```
$ sudo find / -name libboost_filesystem.so*
$ export LD_LIBRARY_PATH=/usr/local/lib/
$ osm2pgsql -c -d japan -S /usr/local/share/osm2pgsql/default.style --host localhost japan-latest.osm.pbf
osm2pgsql SVN version 0.87.2-dev (64bit id space)

Using built-in tag processing pipeline
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads
Allocating memory for dense node cache
Out of memory for node cache dense index, try using "--cache-strategy sparse" instead
Error occurred, cleaning up
```
またエラーになってしまいました。
`--cache-strategy sparse`のオプションをつけよと言われたので、その通りに、、、、
```
$ osm2pgsql -c -d japan -U postgres --cache-strategy sparse -S /usr/local/share/osm2pgsql/default.style --host localhost japan-latest.osm.pbf
osm2pgsql SVN version 0.87.2-dev (64bit id space)

Using built-in tag processing pipeline
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads
Allocating memory for sparse node cache
Node-cache: cache=800MB, maxblocks=102400*8192, allocation method=1
Mid: Ram, scale=100

Reading in file: japan-latest.osm.pbf
Processing: Node(52420k 859.3k/s) Way(0k 0.00k/s) Relation(0 0.00/s)
Node cache size is too small to fit all nodes. Please increase cache size
Error occurred, cleaning up
```
メモリが足りないので`-C`で指定してあげます。

```
$ osm2pgsql -c -d japan -U postgres --cache-strategy sparse -C 2400 -S /usr/local/share/osm2pgsql/default.style --host localhost japan-latest.osm.pbf
osm2pgsql SVN version 0.87.2-dev (64bit id space)

Using built-in tag processing pipeline
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads
Allocating memory for sparse node cache
Out of memory for sparse node cache, reduce --cache size
Error occurred, cleaning up
```
そもそも、メモリが足りない、、、
`-C`の値をいろいろ変えてみても、
メモリが多いとそもそもメモリがないよと怒られ、メモリが少ないとメモリが足りないと言われます、、、、
詰んだ、、、、


ここで諦めてはいけません
[この記事](http://jurishimizu.github.io/2014/12/29/postgis-install-1/)で作った仮想環境のメモリを増やしましょう
まず今のメモリを確認
```
$ free
             total       used       free     shared    buffers     cached
Mem:        469452     117408     352044          0       2432      42320
-/+ buffers/cache:      72656     396796
Swap:       950264      25060     925204
```
vagrantの環境からログアウトして、Vagrantfileにメモリ割当の記述を追加
```Vagrantfile
  config.vm.provider :virtualbox do |vb|
    ・
    ・
    vb.customize ["modifyvm", :id, "--memory", 2048]
  end
```
vagrantを再起動して、メモリを確認
```
$ vagrant reload
$ vagrant ssh
$ free
             total       used       free     shared    buffers     cached
Mem:       1922488     172968    1749520          0       7184      64644
-/+ buffers/cache:     101140    1821348
Swap:       950264          0     950264
```
メモリが増えました。

しかし、この方法で`-C`の値を4200までふやしましたが、
エラーが解消されませんでした、、、、
やっぱり詰んだ、、、

と思っていましたが、slimモードを使うと、メモリが少なくてもいけるそうです。
`--slim`を追加して、再実行！
```
$ osm2pgsql -c -d japan -U postgres --slim -C 4200 --cache-strategy sparse -S /usr/local/share/osm2pgsql/default.style --host localhost japan-latest.osm.pbf
osm2pgsql SVN version 0.87.2-dev (64bit id space)

Using built-in tag processing pipeline
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads
Allocating memory for sparse node cache
Node-cache: cache=4200MB, maxblocks=537600*8192, allocation method=9
Mid: pgsql, scale=100 cache=4200
Setting up table: planet_osm_nodes
Setting up table: planet_osm_ways
Setting up table: planet_osm_rels

Reading in file: japan-latest.osm.pbf
Processing: Node(103668k 1.7k/s) Way(9965k 10.77k/s) Relation(23920 42.64/s)  parse time: 60967s
Node stats: total(103668009), max(3297129224) in 59481s
Way stats: total(9965258), max(322927091) in 925s
Relation stats: total(23920), max(4497829) in 561s
Committing transaction for planet_osm_point
Committing transaction for planet_osm_line
Committing transaction for planet_osm_polygon
Committing transaction for planet_osm_roads
Setting up table: planet_osm_nodes
Setting up table: planet_osm_ways
Setting up table: planet_osm_rels
Using built-in tag processing pipeline

Going over pending ways...
  2856546 ways are pending

Using 1 helper-processes
Finished processing 2856546 ways in 1052 sec

2856546 Pending ways took 1052s at a rate of 2715.35/s
Committing transaction for planet_osm_point
Committing transaction for planet_osm_line
Committing transaction for planet_osm_polygon
Committing transaction for planet_osm_roads

Going over pending relations...
  0 relations are pending

Using 1 helper-processes
Finished processing 0 relations in 0 sec

Committing transaction for planet_osm_point
WARNING:  there is no transaction in progress
Committing transaction for planet_osm_line
WARNING:  there is no transaction in progress
Committing transaction for planet_osm_polygon
WARNING:  there is no transaction in progress
Committing transaction for planet_osm_roads
WARNING:  there is no transaction in progress
Stopping table: planet_osm_rels
Building index on table: planet_osm_rels (fastupdate=off)
Stopping table: planet_osm_ways
Stopping table: planet_osm_nodes
Stopped table: planet_osm_nodes in 0s
Sorting data and creating indexes for planet_osm_point
Sorting data and creating indexes for planet_osm_line
Sorting data and creating indexes for planet_osm_polygon
Building index on table: planet_osm_ways (fastupdate=off)
Sorting data and creating indexes for planet_osm_roads
Analyzing planet_osm_point finished
Stopped table: planet_osm_rels in 10s
Analyzing planet_osm_roads finished
Analyzing planet_osm_polygon finished
Analyzing planet_osm_line finished
Copying planet_osm_point to cluster by geometry finished
Creating geometry index on  planet_osm_point
Copying planet_osm_roads to cluster by geometry finished
Creating geometry index on  planet_osm_roads
Creating osm_id index on  planet_osm_point
Creating indexes on  planet_osm_point finished
All indexes on  planet_osm_point created  in 188s
Completed planet_osm_point
Creating osm_id index on  planet_osm_roads
Creating indexes on  planet_osm_roads finished
All indexes on  planet_osm_roads created  in 216s
Completed planet_osm_roads
Copying planet_osm_polygon to cluster by geometry finished
Creating geometry index on  planet_osm_polygon
Creating osm_id index on  planet_osm_polygon
Creating indexes on  planet_osm_polygon finished
All indexes on  planet_osm_polygon created  in 674s
Completed planet_osm_polygon
Copying planet_osm_line to cluster by geometry finished
Creating geometry index on  planet_osm_line
Creating osm_id index on  planet_osm_line
Creating indexes on  planet_osm_line finished
All indexes on  planet_osm_line created  in 1496s
Completed planet_osm_line
Stopped table: planet_osm_ways in 1873s
node cache: stored: 103668009(100.00%), storage efficiency: 50.00% (dense blocks: 0, sparse nodes: 103668009), hit rate: 100.00%

Osm2pgsql took 63895s overall
```
やっとできました。。。
長い道のりだった、、、、

## データを確認する

ほんとに入ってるか、DBをのぞいてみましょう
```sh
$ sudo su - postgres
$ psql -d japan
```
```sql
japan=# \d
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | geography_columns  | view  | postgres
 public | geometry_columns   | view  | postgres
 public | planet_osm_line    | table | postgres
 public | planet_osm_nodes   | table | postgres
 public | planet_osm_point   | table | postgres
 public | planet_osm_polygon | table | postgres
 public | planet_osm_rels    | table | postgres
 public | planet_osm_roads   | table | postgres
 public | planet_osm_ways    | table | postgres
 public | raster_columns     | view  | postgres
 public | raster_overviews   | view  | postgres
 public | spatial_ref_sys    | table | postgres
(12 rows)
```

テーブルが増えてます！

`planet_osm_polygon`に建物などの「面」の情報（ポリゴン）
`planet_osm_line`に道路などの「線」の情報（ライン）
`planet_osm_point`に「点」の情報（ポイント）が格納されています。
じゃあ、`planet_osm_roads`は何なのかというと、
縮尺が大きいときにつかわれる`planet_osm_line`のサブセットだそうです。
縮尺が大きいときに細い道まで全部表示したら大変だからってことでしょうかね
他の３つは中間テーブルのようです。

詳細は[コチラ](http://wiki.openstreetmap.org/wiki/JA:Osm2pgsql/schema)

テーブルの定義をみてみます

```sql
japan=# \d planet_osm_polygon
           Table "public.planet_osm_polygon"
       Column       |          Type           | Modifiers
--------------------+-------------------------+-----------
 osm_id             | bigint                  |
 access             | text                    |
 addr:housename     | text                    |
 addr:housenumber   | text                    |
 addr:interpolation | text                    |
・・・・・・・
 wetland            | text                      |
 width              | text                      |
 wood               | text                      |
 z_order            | integer                   |
 way_area           | real                      |
 way                | geometry(Geometry,900913) |
```

カラムが多い、、、、
最後にあるgeometryというデータ型がジオなデータなわけですね。
900913というナゾの数字、これは前回のブログでもふれた空間参照系を表しています。
繰り返しになりますが、DBで扱える空間参照系は`spatial_ref_sys`テーブルで管理しています。

```sql
japan=# select * from spatial_ref_sys where srid=900913;
  srid  |       auth_name        | auth_srid |                                                                       srtext                                                                                                       |            proj4text
--------+------------------------+-----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------
 900913 | spatialreferencing.org |    900913 | PROJCS["Popular Visualisation CRS / Mercator (deprecated)",GEOGCS["Popular Visualisation CRS",DATUM["Popular_Visualisation_Datum",SPHEROID["Popular Visualisation Sphere",6378137,0,AUTHORITY["EPSG","7059"]],TOWGS84[0,0,0,0,0,0,0],AUTHORITY["EPSG","6055"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4055"]],UNIT["metre",1,AUTHORITY["EPSG","9001"]],PROJECTION["Mercator_1SP"],PARAMETER["central_meridian",0],PARAMETER["scale_factor",1],PARAMETER["false_easting",0],PARAMETER["false_northing",0],AUTHORITY["EPSG","3785"],AXIS["X",EAST],AXIS["Y",NORTH]] | +proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +no_defs
(1 row)
```

意味不明ですが、[WGS](http://ja.wikipedia.org/wiki/%E6%B8%AC%E5%9C%B0%E7%B3%BB#.E4.B8.96.E7.95.8C.E6.B8.AC.E5.9C.B0.E7.B3.BB.E3.81.A8.E3.81.84.E3.81.86.E5.91.BC.E7.A7.B0)と書いてあるので、世界測地系なのでしょう、、、
wayというカラムはこの空間参照系のデータだよっていうことですね。

テーブルの中身を見てみなしょう。
まずは件数
```sql
japan=# select count(1) from  planet_osm_polygon ;
  count
---------
 2591940
(1 row)
```

カラムも多いけど、データ数も多いですね。日本全国なので、、、、
適当にselectしても混乱しそうなので、データの仕様を確認してみましょう。

OSMの仕様はこちらで確認できます。
http://wiki.openstreetmap.org/wiki/JA:Map_Features

例えば[このページ](http://wiki.openstreetmap.org/wiki/JA:Map_Features#.E5.95.86.E6.A5.AD.E6.96.BD.E8.A8.AD)をみれば、
buildingカラムに「commercial」が設定されていたらは「商業施設なんだな」ってわかるわけですね。
バーとかアイスクリーム屋さんまで指定できるみたいで、眺めてると結構面白いです。

商業施設をselectしてみましょう
```sql
japan=# select name,building from planet_osm_polygon where building='commercial' limit 10 ;
              name               |  building
---------------------------------+------------
 久米島町B&G海洋センター         | commercial
 ヤマダ電機                      | commercial
                                 | commercial
 JPR那覇ビル (JPR Naha Building) | commercial
 和民 (Watami)                   | commercial
                                 | commercial
 A-Price                         | commercial
 西原シティ (Nishihara City)     | commercial
 BEST                            | commercial
                                 | commercial
(10 rows)
```

それっぽい検索結果になったので、ちゃんとデータが入ってるみたいです！
今日はここまで！！！

