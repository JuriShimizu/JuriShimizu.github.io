title: PostGISでGeoなINSERT
date: 2015-03-15 14:23:48
tags: postgis
---

PostGISで地物をINSERTしてみます。

<!-- more -->

## 準備

テスト用にデータベースを用意しておきます。
詳細は[こちら](http://www.jurigis.me/2015/01/09/postgis-install-2/)

```sh
$ sudo su - postgres
$ createdb test
$ psql -d test
```
```sql
test=# create extension Postgis;
CREATE EXTENSION
test=# SELECT Postgis_full_version() ;
                                                                        postgis_full_version

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 POSTGIS="2.1.5 r13152" GEOS="3.4.2-CAPI-1.8.2 r3921" PROJ="Rel. 4.8.0, 6 March 2012" GDAL="GDAL 1.9.2, released 2012/10/08" LIBXML="2.7.6" LIBJSON="UNKNOWN" RASTER
(1 row)
```

## TABLE作成

TABLEを作成します。

GEOMETRYの種類は点・線・面の３種類なので、３つテーブルを用意します。

図形タイプと、SRIDを指定します。
SRIDについては[こちら](http://www.jurigis.me/2015/02/05/about-srid/)を参照

今回は 4326（WGS84、地理座標系）にします。

```sql

test=# CREATE TABLE sample_point (
test(# id SERIAL PRIMARY KEY
test(#  , name text
test(#  , geom GEOMETRY (POINT , 4326)
test(# );
CREATE TABLE
test=#
test=#
test=# CREATE TABLE sample_line (
test(# id SERIAL PRIMARY KEY
test(#  , name TEXT
test(#  , geom GEOMETRY (LINESTRING , 4326)
test(# );
CREATE TABLE
test=#
test=#
test=# CREATE TABLE sample_polygon (
test(# id SERIAL PRIMARY KEY
test(#  , name TEXT
test(#  , geom GEOMETRY (POLYGON , 4326)
test(# );
CREATE TABLE
test=#

```

## INSERT

データをINSERTしてみます。

GEOMETORY型はバイナリなので、
`ST_GeomFromText`を使って、INSERTします。

緯度経度はスペースで区切って、空間参照系を指定します。

まずは**点**をINSERTしてみます。

```sql
test=# INSERT INTO sample_point(name,geom)
test-# VALUES ('point1',ST_GeomFromText('POINT(35.67 139.75)',4326));
INSERT 0 1
```

**線**を指定するときは、緯度経度をカンマで区切ります。

```sql
test=# INSERT INTO sample_line(name,geom)
test-# VALUES ('line1',ST_GeomFromText('LINESTRING(35.67 139.75,35.67 139.755,35.675 139.755 )',4326));
INSERT 0 1
```

**面**を指定するときは、緯度経度の最初と最後を同じにします。

```sql
test=# INSERT INTO sample_polygon(name,geom)
test-# VALUES ('polygon1',ST_GeomFromText('POLYGON((35.68 139.76,35.686 139.76,35.686 139.766,35.68 139.766 ,35.68 139.76 ))',4326));
INSERT 0 1
```

本当にINSERTできてるか、QGISで確認するとこんな感じです。
方法は[こちら](http://www.jurigis.me/2015/01/29/display-postgis-data-in-qgis/)を参照

![QGIS](QGIS_01.png)



