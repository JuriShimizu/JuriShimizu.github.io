title: yumを使わずにwgetでPostGISをインストール
date: 2015-01-09 18:18:40
tags: 
- centos
- postgis
---

[前回](http://jurishimizu.github.io/2015/01/09/postgis-install-2/)はあっさりとyumでPOSTGISをインストールしましたが、
諸事情でyumを使えないこともあるってことで、yumを使わないインストール方法を書いておきます。
postgresqlのインストールは省いてます、、、

<!-- more -->

# PostGISをインストール

[PROJ](http://www.torutk.com/projects/swe/wiki/PROJ4)インストール
```
$ wget http://download.osgeo.org/proj/proj-4.8.0.tar.gz
$ tar xvfz  proj-4.8.0.tar.gz
$ cd proj-4.8.0
$ ./configure --prefix=/opt/geo
$ make
$ make install
```

[GEOS](http://www.informatix.co.jp/top/club2/coding/contents/cont140207-12.html)インストール
```
$ wget http://download.osgeo.org/geos/geos-3.4.2.tar.bz2
$ tar -jxvf geos-3.4.2.tar.bz2
$ cd geos-3.4.2
$ ./configure --prefix=/opt/geo
$ make
$ make install
```

[GDAL](http://ja.wikipedia.org/wiki/GDAL)インストール
```
$ wget http://download.osgeo.org/gdal/1.10.0/gdal-1.10.0.tar.gz
$ tar xvfz gdal-1.10.0.tar.gz
$ cd gdal-1.10.0
$ ./configure --prefix=/opt/geo
$ make
$ make install
```

PostGISインストール
```
$ wget http://download.osgeo.org/postgis/source/postgis-2.0.4.tar.gz
$ tar xvfz postgis-2.0.4.tar.gz
$ cd postgis-2.0.4
$ ./configure --prefix=/opt/geo --with-projdir=/opt/geo --with-gdalconfig=/opt/geo/bin/gdal-config
$ make
$ make install
```

PROJとGEOSのライブラリを /etc/ld.so.conf に登録
```
$ vi /etc/ld.so.conf

下記を追記
/opt/geo/lib
```

あとは
[前回](http://jurishimizu.github.io/2015/01/09/postgis-install-2/)と同じ手順でOKです。

/etc/ld.so.conf を編集を忘れるとこんな感じのエラーになります
```sql
# create extension postgis;
ERROR:  could not load library "/usr/lib/pgsql/postgis-2.0.so": /usr/lib/pgsql/postgis-2.0.so: undefined symbol: pj_get_spheroid_defn
```
PROJのバージョンが4.8より古い場合にも同じエラーになるみたいです。

PostGISを使うときは、PROJとかGDALとか、いろいろ必要なのですね。
yumでインストールしたときは、勝手にインストールしてくれてたってことですね。
yumって便利ですね、、、
