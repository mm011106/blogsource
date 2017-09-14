---
layout: post
title: "Paho-mqttでバイナリファイルを受信してみる"
date: 2015-02-03 21:01:10 +0900
comments: true
categories: phao-mqtt
---

バイナリファイルの受信はmosquitto_subではちょっと面倒じゃないかな、なんてこと言ってましたが、Paho-mqttで簡単なクライアントを作ってバイナリファイルの転送を試して見ました。

<!-- more -->

コードは先の[投稿](http://mm011106.github.io/blog/2014/12/29/vim/)で拾ってきたPaho-mqtt(python)のテストコードをちょちょいと改造しました。

```python test code for binary-file subscription

import paho.mqtt.client as mqtt

# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, rc):
    print("Connected with result code "+str(rc))
  # Subscribing in on_connect() means that if we lose the connection and
  # reconnect then subscriptions will be renewed.
    client.subscribe("topic/binary/jpeg")

# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    # print(msg.topic+" "+str(msg.payload))
    outfile=open('./test.jpg' , 'w')
    outfile.write(msg.payload)
    outfile.close
    

client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

client.connect("my.broker.jp", 1883, 60)

# Blocking call that processes network traffic, dispatches callbacks and
# handles reconnecting.
# Other loop*() functions are available that give a threaded interface and a
# manual interface.
client.loop_forever()

```

これを実行しておいて、別のコンソール（あるいはPC）から、jpgファイルをパブリッシュしてみます。　　
こちらは、mosquitto。
```sh publish a binary file as a test data
$ mosquitto_pub -h my.broker.jp -t topic/binary/jpeg -f mypicture.jpg
``` 

とします。

すると、先程のpythonスクリプトからtest.jpgのファイルが出力されました。実際に表示させてみると、問題なく絵を見ることが出来ました。

mosquitto_subのコマンドラインからですと、出力したファイルはコマンドを終了しない限りずっとオープンしっぱなしなので、スクリプトなどで横取りすることできませんでした。今回のこのpaho版では、ファイルを読み込んだら一回クローズしてしまいますので、横取りできます。

ま、きちんとpythonで全部のスクリプトを書く、というのが筋でしょうけど。


