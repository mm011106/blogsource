---
layout: post
title: "mosquittoをアップデートする"
date: 2015-01-21 20:07:57 +0900
comments: true
categories: [mosquitto, ubuntu]
---
なんか変だと思っていたら、ubuntuに入れたmosquittoのバージョンが最新ではありませんでした。webにあるmanページとどうも違うなと思っていたら、そんなことが原因でした。
なので、アップデート。
<!-- more -->

最初にmosquittoをインストールしたときは、`apt-get install mosquitto-client`でやりましたが、普通にアップデートマネージャでアップデートのチェックをしてもmosquittoは引っかかってきません。

本家の[download page](http://mosquitto.org/download/)を確認すると、最新版はパーソナルパッケージアーカイブ(PPA)からインストールする必要があるとのことでしたので、指示のとおりインストールします。  
私、PPAは初耳でした。

こちらも参照。
[mosquitto PPA team](https://launchpad.net/~mosquitto-dev/+archive/ubuntu/mosquitto-ppa)


```sh commandlines for upgarde mosquitto and client.
$ sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
$ sudo apt-get update
$ sudo apt-get dist-upgrade
#  dist-upgrade は　upgrade するときにコンフリクトする（あるいは不要な）前のパッケージを削除します。
```

おっと、 dist-upgarde ではブローカのconfigファイルは__上書きされてしまいます__！
必要に応じてバックアップを取ってください！！！！！  
client だけをインストールしている場合は関係ないです。

きちんとインストールできたか、`mosquitto_sub --help`で確認してみます。

```
mosquitto_sub is a simple mqtt client that will subscribe to a single topic and print all messages it receives.
mosquitto_sub version 1.3.5 running on libmosquitto 1.3.5.

Usage: mosquitto_sub [-c] [-h host] [-k keepalive] [-p port] [-q qos] [-R] -t topic ...
  :
  :
```

とでてきましたので、多分大丈夫。

これで、-Nオプションが使えるはず。

ちなみに、Raspberry Pi用のビルドは普通に`apt-get install` でインストールしても最新版が入るようです。
