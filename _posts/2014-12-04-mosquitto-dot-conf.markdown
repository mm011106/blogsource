---
layout: post
title: "mosquitto.confを見てみた"
date: 2014-12-04 20:21:14 +0900
comments: true
categories: [MQTT, mosquitto.conf]
---

やっぱりmanは見ないとダメ、ということで、まずは`mosquitto.conf`を紐解いています。
原文はこちら[`man mosquitto.conf(5)`](http://mosquitto.org/man/mosquitto-conf-5.html)

つらつらとながめていると、昨日疑問に思っていた部分が書かれていました。
<!-- more -->

**mount_point**  *topic prefix*  

> This option is used with the listener option to isolate groups of clients. 
> When a client connects to a listener which uses this option, the string argument is attached to the start of all topics for this client. This prefix is removed when any messages are sent to the client. This means a client connected to a listener with mount point *__example__* can only see messages that are published in the topic hierarchy *__example__* and above.

さらに、`mosquitto.conf`のデフォルトでは、この部分が

```
# The listener can be restricted to operating within a topic hierarchy using
# the mount_point option. This is achieved be prefixing the mount_point string
# to all topics for any clients connected to this listener. This prefixing only
# happens internally to the broker; the client will not see the prefix.
#mount_point
```
となっています。

「クライアント」と「リスナー」という言葉が出てきて？？？な感じです。。。。

> The network ports that mosquitto listens on can be controlled using listeners.

とあるので、ブローカーがパブリッシャからの聞き耳を立てるポートをコントロールするためのものが「リスナー」なんでしょうか。どうも、そうみたいですね。
「リスナー」は、MQTTのプロトコルを通信するためのポートに、この設定ファイルで設定できるようなメソッドを組み込んだオブジェクトみたいな感じでしょうか。

その「リスナー」の部分、`mosquitto.conf`では

``` 
# Listen on a port/ip address combination. By using this variable 
# multiple times, mosquitto can listen on more than one port. If 
# this variable is used and neither bind_address nor port given, 
# then the default listener will not be started.
# The port number to listen on must be given. Optionally, an ip 
# address or host name may be supplied as a second argument. In 
# this case, mosquitto will attempt to bind the listener to that 
# address and so restrict access to the associated network and 
# interface. By default, mosquitto will listen on all interfaces.
# listener port-number [ip address/host name]
#listener
```

となっているので、ポートを「リスナー」として設定して開けば、その一つ一つに個別のマウントポイントを設定できる、という感じですかね。でもサブスクライバの方はどうなるんでしょ？サブスクライブできるトピックを限定できるのかなあ。そうか、ポート毎（リスナー毎）にマウントポイントを設定できればそのポート（リスナー）のサブスクライバは全部そのマウントポイントしか見えない、ってことか。パブリッシャも同様。
listenerごとにipアドレスやホストネームをバインドできるのはそのためですかね。とすれば、IPアドレスは複数設定できなきゃいけませんね。。。。

その他セキュリティ設定のところでも、特定のトピックの読み書きが可能なように設定できたりするようです。

そもそも「クライアント」はサブスクライバとパブリッシャの両方のことを意味するのかしら？？？

奥深し。。。。。

