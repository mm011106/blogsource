---
layout: post
title: "Paho mqttでリモートカメラ"
date: 2015-02-08 09:49:46 +0900
comments: true
categories: Paho-mqtt, webcam, raspi  
---

Paho MQTTでリモートでシャッターを切れるカメラを作ってみました。撮った写真もMQTTで送られてきます。

> 2015/2/28 UPDATE!! 

<!-- more -->
##カメラ側の設定

まずは、デバイス（カメラ）側のスクリプト

ハードウエアとして、Raspberry Pi B+に専用のカメラモジュールをつけています。

```python remotecam.py 
#!/usr/bin/env python
#
#

import paho.mqtt.client as mqtt
import os

# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, rc):
    print("Connected with result code "+str(rc))
  # Subscribing in on_connect() means that if we lose the connection and
  # reconnect then subscriptions will be renewed.
    client.subscribe("my/device/stillcam/command")

# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    cmd = str(msg.payload)
    print(msg.topic+" "+str(msg.payload))
    if cmd == "shoot":
        print "Say cheeees!"

        dummy = os.system("raspistill -w 1024 -h 768 -t 10 -o /run/shm/temp.jpg")
        dummy = os.system("mosquitto_pub -h my.broker.jp -t my/device/stillcam -f /run/shm/temp.jpg")

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

パブリッシュ設定したメッセージが来ると、on_messageがコールされます。  
メッセージが"shoot"だったら、osコマンド"raspistill"を実行して写真を撮ります。  
さらにそのデータをosコマンド"mosquitto_pub"でパブリッシュします。

ちょっと格好悪いですけど、MQTTで写真データを送るのにmosquittoを使っています。~~ざっくり試した感じですと、paho-mqttではバイナリのペイロードをうまくハンドリングできないようで、多分なんか設定があるのだと思います。探してみます。~~

#### 2015/2/11追記：

####上記のバイナリデータのハンドリングの件ですが、うまく行かない理由は私のスキル不足です。多分python内部での変数のデータ扱いをうまく変換してやる必要が有りそうです。  只今勉強中。

> ### 2015/2/28  追記：
> 上記、バイナリデータの送信の件ですが、  
> 実際に出てくるエラーは  
> `UnicodeDecodeError: 'ascii' codec can't decode`  
> というものでしたので、そのまま検索をかけてみると、デコードがうまく行っていない、旨のエラーのようです。  
> であれば、変換をしないように設定すればいいのかなあ、どうするのかなあ、と思いつつソースコードを（わからないなりに）眺めていると、

```sh http://git.eclipse.org/c/paho/org.eclipse.paho.mqtt.python.git/tree/src/paho/mqtt/client.py
            if isinstance(payload, str):
                upayload = payload.encode('utf-8')
                payloadlen = len(upayload)
            elif isinstance(payload, bytearray):
                payloadlen = len(payload)
            elif isinstance(payload, unicode):
                upayload = payload.encode('utf-8')
                payloadlen = len(upayload)
```

> という記述を見つけ、payloadのタイプによってエンコードを分けていることがわかりました。`bytearray`なら何もせずにそのままのデータが送られるので`bytearray`にすればいい。   
> 再び検索して、読み込んだファイルをbytearray型で変数に代入する方法を調べ、結果として次のようなソースコードになりました。 

```sh replace with 'mosquitto_pub ......'
# 撮影した写真ファイルは/run/shm/temp.jpg
       with open('/run/shm/temp.jpg', 'rb') as source:
            payload_pub = bytearray(source.read())

# bytearray 型にして変数に代入、それをそのままペイロードとしてパブリッシュ
        client.publish(topic_root+topic_pub, payload_pub)
```
> これで無事バイナリファイルを送信することが出来ました。


USBのカメラを使う場合はraspistillのコマンドを適宜変更すればいいかと思います。

このスクリプトを実行可能に設定して、実行させます。これで待ち受け状態。

##コントローラ側

別のPCでは、

- シャッターを切るコマンドを発行する
- 送られてきた写真のデータを保管する

という作業があります。

まずは、送られてきたデータを保管するスクリプトを作ります。

```python photo_sub.py
#!/usr/bin/env python
#
#
#

import paho.mqtt.client as mqtt
import datetime

# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, rc):
    print("Connected with result code "+str(rc))
  # Subscribing in on_connect() means that if we lose the connection and
  # reconnect then subscriptions will be renewed.
    client.subscribe("my/device/stillcam")

# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    
    # print(msg.topic+" "+str(msg.payload))
    filename = "./image/" + datetime.datetime.today().strftime("%H%M%S%f") + ".jpg"
    outfile=open( filename , 'w')
    outfile.write(msg.payload)
    print "subscribe: " + filename
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

このスクリプトでは、my/device/stillcamというトピックにパブリッシュされたデータを取り込んでファイル名（タイムスタンプ）をつけて保存します。

このスクリプトを実行して、データを待ち受けます。

最後にシャッタを切るコマンドを送ります。  
とりあえずは、コマンドラインからです。

```sh 
$ mosquitto_pub -h my.broker.jp -t my/device/stillcam/command -m "shoot"
```

これで、メッセージ"shoot"をトリガにして、写真を撮影し、それを転送して保存するまでの一連の作業が行われます。

デバイス側の処理負荷を確認するため1秒に1回シャッターを切る動作を続けて見ました。
結果的には、カメラと転送の処理で数%程度の負荷のようです。  
ちなみにこのカメラ、この設定では1秒間隔以上のスピードで連続して撮影することはできませんでした。

カメラがRaspi用のモジュールですので、かなりオーバーヘッドが小さい感じもします。
CPU的には処理が軽くていいのですが、ハードウエア的には取り回しが悪く、ちょっといまいちな感じもします。  
応用のシーンによりますが、今回私が想定しているケースでは使いにくいです。

あとで、USBcamでやってみようと思います。


