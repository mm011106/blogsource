---
layout: post
title: "自分のIPアドレスを送信する"
date: 2015-01-13 20:43:51 +0900
comments: true
categories: [Raspi, ipconfig]
---

自分のIPアドレスをMQTTで送信してもらうと、デバグの時にIPアドレスをいちいち調べなくていいので楽だなあ、とおもい試して見ました。
<!-- more -->

Raspberry Pi で、まずは、どうやってIPアドレスを抽出するかです。

自分のIPアドレスは`ifconfig`で出てきますが、これは人間用なのでフォーマットがややこしい。

```sh example of result of "ifconfig" command
$ ifconfig eth0

eth0      Link encap:Ethernet  HWaddr b8:27:eb:29:61:2d  
          inet addr:192.168.0.xyz  Bcast:192.168.0.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:95729 errors:0 dropped:67 overruns:0 frame:0
          TX packets:62090 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:10362519 (9.8 MiB)  TX bytes:6910384 (6.5 MiB)

```

色々と悩んだ末、こんな感じのコマンドラインではどうかと。。。。

```sh a example of one-liner extracting IP adress from "ifconfig" command
ifconfig eth0 | grep -o '^ *inet addr:.*B' | grep -E -o '([0-9]{1,3}\.){3}[0-9]{1,3}'
```

最初のgrepでインターフェイスのIPアドレスを表示しているフレーズだけを抜き出します。  
具体的には行頭から任意の数のスペースがあって、"inet addr:"とあって、任意の文字が続き"B"で終わる部分です。

`grep -o '^ *inet addr:.*B'`

次に、そのフレーズの中には1つしかIPアドレスは入っていないはずなので、それを抜き出します。

`grep -E -o '([0-9]{1,3}\.){3}[0-9]{1,3}'`

だいたいこれで、行けそうです。
試しにやってみると。

```sh test for the one-liner 
$ ifconfig eth0 | grep -o '^ *inet addr:.*B' | grep -E -o '([0-9]{1,3}\.){3}[0-9]{1,3}'
192.168.0.xyz
```

今回の目的には十分かとおもいます。

しかし、なんかもっといい方法（どっかのファイルを見るとか）がありそうなきもします。。。。

----

#### 2015/02/19 追記
> 今日、IPアドレスを表示するコマンドを発見しました。

> ` $ hostname -I`

> なんで検索しなかったんだろう。。。。

----

さて、このコマンドラインをデバイスのスクリプト（cronで動く）にいれてみたところ、うまく動かない。具体的には、結果がNULLになってしまうのです。試しに、手元でコマンドラインから実行するとうまく動く。うう〜ん。  
さっぱり理由が分からず、いろいろ調べてみたところ「[動かないときはエラーログを取れ！](http://higelog.brassworks.jp/?p=1775)」という啓示をいただき、試してみたところ、`command not found`とのこと。ifconfigコマンドへのパスが通ってない、という情けないという状況。言われてみれば確かにそんなことをどこかで見たような気がするなあ。。。。

ということで、ifconfigコマンドを絶対パスで指定して事なきを得ました。

__「CRONのコマンドは絶対パス指定」__　勉強しました。。。



