---
layout: post
title: "disable-clean-sessionオプション"
date: 2014-12-05 21:14:40 +0900
comments: true
categories: [MQTT, mosquitto_sub]
---

一昨日から気になっていた[`mosquitto_sub --disable-clean-session`](http://mosquitto.org/man/mosquitto_sub-1.html)オプションですが、試してみました。

ブローカに向かって`--desiable-clean-session`と`--id`の2つのオプションをつけてサブスクライブを要求します。
<!-- more -->

```sh
$ mosquitto_sub -h HOSTNAME -t YOUR/TOPIC/NAME --qos 1 --disable-clean-session --id CONNECTION-ID

1417781342, 1691, 0, 0, 0
1417781402, 1691, 0, 0, 0
1417781461, 1691, 0, 0, 0
1417781522, 1691, 0, 0, 0　　<<時刻 21:12:02　で中断
^C
```
`--id`オプションはサブスクライバが同一であることを主張するためのもののようですので、これが一致することで「再接続だから配信していないデータを送らなきゃ」と認識してくれます。`--id`を指定しないとデフォルトのプロセス番号を元にしたものになるので、プロセスを止めて再度起動した場合「再接続」と認識してもらえません。
また、--qosが1もしくは2でないとダメみたいです。QoS　0は配信保証がなされないクオリティ指定ですので、当然といえば当然。

データが来たところで、適当に中断します。このトピックではパブリッシャが1分毎に1データを送ってきます。データの最初のフィールドがエポックタイムです。よく見ると60づつ増えていっているのがわかります。

そしてCtrl-Cでプロセスを止めます。

では、ここでしばらく待って。。。。。

わかりやすいように、タイムスタンプを印刷してからサブスクライブを再開します。

```sh
$ date ; mosquitto_sub -h __host-name__ -t YOUR/TOPIC/NAME --qos 1 --disable-clean-session --id  testclient

Fri Dec  5 21:24:54 JST 2014
1417781581, 1692, 0, 0, 0　　<<　再開したのは21:24:54だけど、21:13:01のデータから配信されている。
1417781642, 1691, 0, 0, 0
1417781701, 1691, 0, 0, 0
1417781761, 1692, 0, 0, 0
1417781822, 1691, 0, 0, 0
1417781881, 1691, 0, 0, 0
1417781942, 1692, 0, 0, 0
1417782002, 1692, 0, 0, 0
1417782061, 1692, 0, 0, 0
1417782121, 1692, 0, 0, 0
1417782182, 1692, 0, 0, 0
1417782241, 1692, 0, 0, 0
1417782302, 1692, 0, 0, 0
1417782361, 1692, 0, 0, 0
   ：
   :
```
おお、きちんと中断している間のデータも配信されました。

`--id`オプションには次のように書かれています。


** -i, \- \-id **

> The id to use for this client. If not given, defaults to mosquitto_sub\_ appended with
> the process id of the client. Cannot be used at the same time as the ** _--id-prefix_ ** argument.

通常、IDは`mosquitto_sub_${PID}`になるようです。でもpidは起動するたびに変わってしまうので、同じクライアントとは認識されませんね。物理経路のトラブル等で通信が途絶えた場合は大丈夫でしょうけれど。

`--id-prefix`はプリフィックスを固定してしまうオプションなので、`--id`とは共存できないということですね。

関連するところで  
ブローカの方の設定で`clientid_prefixe`というのがあるのですが、これを設定（mosquitto.conf内）すると、指定したプリフィックスのついたクライアントしか接続出来ないようにできるようです。

__clientid_prefixes__ _prefix_
>  
> If defined, only clients that have a clientid with a prefix that matches clientid_prefixes will be
> allowed to connect to the broker. For example, setting _"secure-"_ here would mean a client 
> "_secure-_client" could connect but another with clientid "mqtt" couldn't. By default, 
> all client ids are valid.

となるようです。セキュリティの一助になるかしら？


