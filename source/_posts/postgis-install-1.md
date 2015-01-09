title: POSTGISの環境を構築　その１
date: 2014-12-29 17:40:17
tags: POSTGIS
---

まずは[POSTGIS](http://ja.wikipedia.org/wiki/PostGIS)をインストールしてみます。
POSIGISを入れるとPostgreで地理空間情報が扱えるようになってしまうのです。
データベースで地理情報を扱えるなんてビックリです。

# 仮想環境を作ってみる

まずは、Vagrant と VirtualBox を使って、仮想環境を作ってみます。
自分のPCにPOSTGISをインストールしてもいいのですが、失敗した時のことを考えて、、、

## Vagrant をインストール

こちらのサイトからdmgファイルをインストール

[Download Vagrant – Vagrant](http://www.vagrantup.com/downloads.html)

インストール後、バージョン確認

``` sh
$ vagrant --version
Vagrant 1.7.1
```

## VirtualBox をインストール

こちらのサイトからdmgファイルをインストール

[Downloads – Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Launcpad に VirtualBox のアイコンがあればOK

## 仮想マシンを作成

まずはboxを確認

``` sh
$ vagrant box list
centos65        (virtualbox, 0)
chef/centos-6.5 (virtualbox, 1.0.0)
precise32       (virtualbox, 0)
```

Vagrantfileを作成
``` sh
$ mkdir -p ./Vagrant/CentOS65
$ cd ./Vagrant/CentOS65
$ vagrant init chef/centos-6.5
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
$ ls
Vagrantfile
```
作成されたVagrantfileの中身を確認して、`config.vm.box = "chef/centos-6.5"`となっていれば大丈夫。

VMを起動する
``` sh
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'chef/centos-6.5'...
・・・
```

## ネットワーク設定をする

Vagrantfileの下記の記述のコメントアウトを削除
``` 
# config.vm.network "private_network", ip: "192.168.33.10"
``` 

vagrant のリロード
``` sh
$ vagrant reload
==> default: Attempting graceful shutdown of VM...
==> default: Checking if box 'chef/centos-6.5' is up to date...
・・・
```

これで192.168.33.10 というIPでアクセスできるようになりました
``` sh
$ ping 192.168.33.10
PING 192.168.33.10 (192.168.33.10): 56 data bytes
64 bytes from 192.168.33.10: icmp_seq=0 ttl=64 time=0.585 ms
64 bytes from 192.168.33.10: icmp_seq=1 ttl=64 time=0.583 ms
```

## 仮想環境に接続

``` sh
$ vagrant ssh
Last login: Mon Dec 29 09:03:57 2014 from 10.0.2.2
[vagrant@localhost ~]$
```

これで、仮想環境の準備はOK！

というわけで、今日はここまで、、、、
Geoな話までいかなかった、、、、

