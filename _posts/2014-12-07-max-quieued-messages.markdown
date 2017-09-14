---
layout: post
title: "max_queued_messagesオプションの動作"
date: 2014-12-07 10:14:05 +0900
comments: true
categories: [MQTT, mosquitto.conf, max_queued_messages]
---

ということで、先の`max_queued_messages`オプションを試してみました。

結論から言うと、バッファリングしているのは__指定した数の最も古いデータ__でした。

<!-- more -->

確認手順：

- mosquitto.confに`max_queued_messages 30`と設定  
これで、30個のデータが保持されるはず。

- ブローカが動いている計算機から `$ sudo /etc/init.d/mosquitto restart`　で設定を反映

- QoS１、clear_sessionをfalseでブローカにサブスクライブ要求します。

- 適当に中断

- 適当に再開  
今回は寝る前に止めて朝起きて再接続してみました。  
あとでデータを確認できるよう`reconnect.txt`というファイルに保存。データの第一フィールドは送信時のタイムスタンプになっています。

で、送られたデータを確認してみると。。。。

```sh
$ cat reconnect.txt | awk -F , '{ command = "date -d @" $1 ; printf NR"   " ; system(command) }'
1   Sat Dec  6 21:33:01 JST 2014
2   Sat Dec  6 21:34:02 JST 2014
3   Sat Dec  6 21:35:01 JST 2014
  (...snip..)
27   Sat Dec  6 21:59:01 JST 2014
28   Sat Dec  6 22:00:01 JST 2014
29   Sat Dec  6 22:01:01 JST 2014
30   Sat Dec  6 22:02:02 JST 2014 <指定した数だけ古いデータがあって
31   Sun Dec  7 09:57:01 JST 2014 <そのあと最新の配信データが届く
32   Sun Dec  7 09:58:02 JST 2014
```

ということで、停止してすぐのデータが指定数だけ保存されている、という結論です。

