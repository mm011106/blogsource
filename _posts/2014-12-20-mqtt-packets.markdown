---
layout: post
title: "MQTTのパケットを覗いてみた(1)"
date: 2014-12-20 12:24:00 +0900
comments: true
categories: [mqtt, wireshark]
---

mosquitto_pubなど、mosquittoのクライアント用コマンドで接続した時、どんなことになっているのかちょっと気になったので調べて見ることにしました。
ここでは、サブスクライブ要求した時にどんなことが行われているか、また、mosquitto_subをkillで殺した時、なにか通信しているのか、ということを確認しました。
<!-- more -->

## mosquitto_sub でサブスクライブ要求した場合

まずは、パケットをキャプチャするものを用意しなければいけません。
定番、ということでwiresharkをインストールして、キャプチャ用のソフトにデバイスへのアクセスのパーミッションを付与します。
[こちらのページ](http://www005.upp.so-net.ne.jp/develop-tom/deb/wireshark-deb.html)にわかりやすく記載されています（ありがとうございます）。

まとめておくと、
```sh Install wireshark and configure...
:# apt-get install wireshark
:# setcap 'CAP_NET_RAW+eip CAP_NET_ADMIN+eip' /usr/bin/dumpcap
```

キャプチャインターフェイスをeth0にし、IPアドレスのフィルタにブローカのアドレスを指定してキャプチャを始めます。

サブスクライブを要求します。

	:$ mosquitto_sub -v -t \$SYS/broker/messages/# -h mqtt.broker.host.name


MQTTプロトコルメッセージの最初2バイトは固定ヘッダーになります。MQTTプロトコルメッセージの最小単位です。

TCPレベルでの接続作業があって、MQTTの接続が始まります。

---

### STEP 1: 接続要求　Client -> Broker

```sh 
0000   10 22 00 06 4d 51 49 73 64 70 03 02 00 3c 00 14  ."..MQIsdp...<..
0010   6d 6f 73 71 5f 73 75 62 5f 33 35 38 35 5f 75 62  mosq_sub_3585_ub
0020   75 6e 74 75                                      untu
```

- 固定ヘッダ：  
0000 : メッセージタイプ (メッセージ’１’＝CONNECT）  
0001 : 長さ (続く可変ヘッダーのメッセージ長、ここでは34byte)  

- 可変ヘッダー  
0002 : 次のメッセージの長さ　MSB  
0003 : LSB　（ここでは6byte）  
0004 : "MQIsdp" （ここは定型句のようです）  
000A : プロトコルバージョン "03"  
000B : FLAGs (CleanSession='1')  
000C : Keep Alive timer[s] MSB  
000D : Keep Alive timer[s] LSB  （ここでは60秒）  
000E : 次のメッセージの長さ　MSB  
000F :  　　LSB　（ここでは20byte)   
0010 : クライアント識別子 (オプション--idで指定したものが入ります。この例ではデフォルト値）

まず、ブローカにMQTTプロトコルの接続要求をだします。自分のIDを指定していますので、ブローカは誰が何のトピックをサブスクライブしているか、をこのときのIDをキーにして把握します。

----

### STEP 2 : 要求の返事　Client <- Broker

```
0000   20 02 00 00                                       ...
```

- 固定ヘッダ：  
0000 : メッセージタイプ (メッセージ’2’＝CONNACK）  
0001 : 長さ 2byte

- 可変ヘッダ：  
0002 : トピック名圧縮要求(予約：未使用）  
0003 : 接続要求戻りコード（ここでは'00'で接続許可。'00'以外は接続拒否）

先の接続要求に対して、許可の返事がブローカから来ます。

----


### STEP 3: サブスクライブ要求　Client -> Broker

```
0000   82 1b 00 01 00 16 24 53 59 53 2f 62 72 6f 6b 65  ......$SYS/broke
0010   72 2f 6d 65 73 73 61 67 65 73 2f 23 00           r/messages/#.
```

- 固定ヘッダ：  
0000 メッセージタイプ (メッセージ’8’＝SUBSCRIBE, このメッセージ自身をQoS=1で送信）  
0001 長さ　27byte

- 可変ヘッダ：  
0002 メッセージID　16bit (MSB,LSB)  
0004 ペイロード長さ　16bit (MSB,LSB) ここでは22byte  
0005 ペイロード本体　ここではサブスクライブ要求するトピックを指定。  
001C QoS指定　要求するサブスクライブのQoS指定　QoS='0'  

サブスクライブ要求をします。同時にどのトピックを購読するのかを指定します。

複数トピックを指定した場合は、可変ヘッダに「ペイロード長さ指定+ペイロード本体（トピック名）+QoS指定」が続きます。

---

### STEP 4: データの送付　Client <- Broker

本当は、ここでSUBACKがブローカから送られてくるはずなのですが、それが無く、いきなりデータが送られてきました。

```
0000   31 1f 00 1b 24 53 59 53 2f 62 72 6f 6b 65 72 2f  1...$SYS/broker/
0010   6d 65 73 73 61 67 65 73 2f 73 74 6f 72 65 64 37  messages/stored7
0020   32 31 24 00 1d 24 53 59 53 2f 62 72 6f 6b 65 72  21$..$SYS/broker
0030   2f 6d 65 73 73 61 67 65 73 2f 72 65 63 65 69 76  /messages/receiv
0040   65 64 37 33 38 33 30 31 20 00 19 24 53 59 53 2f  ed738301 ..$SYS/
0050   62 72 6f 6b 65 72 2f 6d 65 73 73 61 67 65 73 2f  broker/messages/
0060   73 65 6e 74 35 39 34 34 31                       sent59441
```

固定ヘッダ：  
0000 : メッセージタイプ (メッセージ’3’＝PUBLISH, このメッセージ自身をQoS=1で送信）  
0001 : 長さ　31byte

可変ヘッダ：
0002 : トピック名長さ　16bit (MSB,LSB) ここでは27byte  
0004 : トピック名　'$SYS/broker/messages/stored'  
  (QoSによっては、ここにメッセージID（16bit）が入る）  
001F : ペイロード　上記トピックのデータが来る  
　（ここまでが固定ヘッダで指定された長さ）

 複数トピックが同時に同じパケットで配信される場合、「固定ヘッダ(PUBLISHメッセージ)+可変ヘッダ(トピックとデータ）」が続きます。

この例でも、0021から他のトピックのデータが繰り返されているのがわかります。

```
0020      31 24 00 1d 24 53 59 53 2f 62 72 6f 6b 65 72  .1$..$SYS/broker
0030   2f 6d 65 73 73 61 67 65 73 2f 72 65 63 65 69 76  /messages/receiv
0040   65 64 37 33 38 33 30
```



---

よくわからなかったのが、固定ヘッダの1バイトでどうやって長いメッセージの長さを表現するのか、というところです。

仕様をよく見ると下記のように書いてありました。
>The variable length encoding scheme uses a single byte for messages up to 127 bytes long. Longer messages are handled as follows.  

> Seven bits of each byte encode the Remaining Length data, and the eighth bit indicates any following bytes in the representation.  
> Each byte encodes 128 values and a "continuation bit".  
> For example, the number 64 decimal is encoded as a single byte, decimal value 64, hex 0x40.  
> The number 321 decimal (= 65 + 2*128) is encoded as two bytes, least significant first. The first byte 65+128 = 193. Note that the top bit is set to indicate at least one following byte. The second byte is 2.

この日本語訳もよくわからないし、英語も何が言いたいのかよくわからずに、仕様書にある例をつらつら眺めると。。。
つまりは、

- 全体の長さをバイナリで示して、下から7ビット切り離す
- 切った残りのbitに'1'がのこっているのなら、切り取った7bitの最上位に'1'をつけて、最初の「長さを示す値(1byte)」とする
- さらに残ったものを同じように処理して、残りが0になるまでやる

という事がわかりました。

仕様では、長さを示す数値は4バイトまで長くできる、ということなので
単純に`7 x 4 = 28 bit`長の変数(unsigned int)だ、ということです。 

固定長でフィールドを用意するのではなく、短いメッセージをできるだけ効率良く送出するために、こんな工夫をしているのだなあと思った次第。

---

mosquitto_subを起動した状態から、killあるいはCtrl-Cでプロセスを止めた場合ですが、

これは、ご想像通り、TCPのセッションは切れますが、MQTTプロトコル上でのUNSUBSCRIBEはなされません。ですので、同じIDを指定して接続した場合、再度同じトピックが配信されます。
しかし、mosquittoのクライアントでは、単純に接続する、というだけのコマンドが無いので、新たにトピックをサブスクライブする指定しかできません。その場合、前のサブスクライブでclean sessionをどのように指定していたか、で結果が変わります。

clean sessionでサブスクライブしていたもの（mosquitto_subではこれがデフォルト）はセッションが切れて、新たにセッションが開始されます。  
clean sessionをDisable していたセッションも、あらたに発行したサブスクライブセッションがclean sessionだと、以前のセッションは切断されます。  
最初のサブスクライブも次のサブスクライブもclean sessionをdisableしていると両方のサブスクライブセッションが維持されて（多分、一つにマージされる）、両方のデータが配信されます。

今回、パケットを確認してみて、mosquittoのクライアントソフトではあまり行儀のいいクライアントスクリプトは書けないなあ、という感じがしました。
デバイス側はRaspiからmbedに移行しようという計画なので、その時にいいクライアントをつくりましょ。





