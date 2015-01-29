title: CentOSにPostgreSQL/PostGISの環境構築
date: 2015-01-09 11:46:17
tags: 
- vagrant
- centos
- virtualized_environment
- postgresql
- postgis
---

[前回](http://jurishimizu.github.io/2014/12/29/Postgis-install-1/)作った仮想環境に、
[PostGIS](http://ja.wikipedia.org/wiki/Postgis)をインストールしてみます。
PostGISを入れるとPostgreSQLで地理空間情報が扱えるようになってしまうのです。
データベースで地理情報を扱えるなんてビックリです。

<!-- more -->

# PostgreSQL/PostGISをインストール

前回作成した仮想環境に接続して、Postgisをインストールします

仮想環境に接続
``` 
$ vagrant ssh
```
PostgreSQLとPostGISインストール
``` 
$ rpm -ivh http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-1.noarch.rpm
$ yum -y install postgresql93 postgresql93-server postgresql93-libspostgresql93-contrib postgresql93-devel
$ rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ yum -y install Postgis2_93
```
バージョン確認
```
$ psql --version
psql (PostgreSQL) 9.3.5
```
自動起動をonにする
```
$ chkconfig postgresql-9.3 on
$ chkconfig --list postgresql-9.3
postgresql-9.3 	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```
DB初期化
```
$ service postgresql-9.3 initdb
Initializing database:                                     [  OK  ]
```
外部からのアクセスを許可するように設定を変更する
postgresql.conf編集
```
$ vi /var/lib/pgsql/9.3/data/pg_hba.conf
 
 変更前
 host    all             all             127.0.0.1/32            ident
 変更後
 host    all             all             0.0.0.0/0            trust
```
pg_hba.conf編集
```
$ vi /var/lib/pgsql/9.3/data/pg_hba.conf

 変更前
 # listen_addresses = ‘localhost’
 # port = 5432
 変更後
 listen_addresses = ‘*’
 port = 5432
```
サーバ起動
```
$ service postgresql-9.3 start
Starting postgresql-9.3 service:                           [  OK  ]
```
## Geoなデータベースを作成

まずは普通にデータベースを作成
後で日本地図のデータを入れたいので、名前はjapanにします
```
$ sudo su - postgres
$ createdb japan
```
ログインして`create extension Postgis;`を実行します
```
$ psql -d japan
```
```SQL
japan=# create extension Postgis;
CREATE EXTENSION
japan=# SELECT Postgis_full_version() ;
                                                                        Postgis_full_version

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Postgis="2.1.5 r13152" GEOS="3.4.2-CAPI-1.8.2 r3921" PROJ="Rel. 4.8.0, 6 March2012" GDAL="GDAL 1.9.2, released 2012/10/08" LIBXML="2.7.6" LIBJSON="UNKNOWN" RASTER
(1 row)
```
これでGeoなデータベースの出来上がりです！

ここでデータベースの中身を確認してみます
```SQL
japan=# \d
               List of relations
 Schema |       Name        | Type  |  Owner
--------+-------------------+-------+----------
 public | geography_columns | view  | postgres
 public | geometry_columns  | view  | postgres
 public | raster_columns    | view  | postgres
 public | raster_overviews  | view  | postgres
 public | spatial_ref_sys   | table | postgres
(5 rows)
```
まだ何もしてないのに、テーブルとビューができてます。
このおかげでGeoなデータが扱えるようになるみたいですね。

`spatial_ref_sys`の中身を除いてみます。
```SQL
japan=# \d spatial_ref_sys
         Table "public.spatial_ref_sys"
  Column   |          Type           | Modifiers
-----------+-------------------------+-----------
 srid      | integer                 | not null
 auth_name | character varying(256)  |
 auth_srid | integer                 |
 srtext    | character varying(2048) |
 proj4text | character varying(2048) |
Indexes:
    "spatial_ref_sys_pkey" PRIMARY KEY, btree (srid)
Check constraints:
    "spatial_ref_sys_srid_check" CHECK (srid > 0 AND srid <= 998999)

```
このテーブルはデータベース中で利用できるすべての[空間参照情報](http://www.geopacific.org/opensourcegis/gcngisbook/QGIS_book/7b2c17ae0/7b2c17ae07b2c37bc062955f716cd530686e2c57307cfb)を定義しています。
空間参照情報って何？？？って感じですが、要はその空間情報がどういう基準で作成されているか？ってことだと思います。
（たぶんもっともっと奥深いものですが、まだあまり理解できてません、、、）

この辺はもうちょっと勉強してから記事を書きたいと思います。

次回は、このデータベースにデータを入れていきたいと思います！！！


