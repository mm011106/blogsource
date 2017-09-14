---
layout: post
title: "mosquitto_subのオプション"
date: 2014-12-03 20:43:00 +0900
comments: true
categories: [MQTT, mosquitto_sub]
---

今日も何気なく検索しながら、webを眺めていたら、`mosquitto_sub`のこんなオプションを発見。

**`-c, --disable-clean-session `**
<!-- more --> 


`Disable the 'clean session' flag. This means that all of the subscriptions for the client will be maintained after it disconnects, along with subsequent QoS 1 and QoS 2 messages that arrive. When the client reconnects, it will receive all of the queued messages.`
`If using this option, it is recommended that the client id is set manually with --id`

これがちゃんと動けば、結構いいですよねえ。でもまあ、ちゃんと`man`読めよ、って話ですな。

週末に試してみよう。

ところで全く話は違いますが、`mosquitto(8)`では1つのブローカのプロセスで1つのトピックツリーを持っているのですよね。
もし、1つのブローカを多くのユーザが共有した場合、お互いのトピックツリーは自由に行き出来できてしまいますよねえ。多くのユーザで使いたいときはどうすればいいのかなあ、と思っていた次第。
`-iオプション`でユーザ名を指定すればユーザ名ごとにトピックツリーを作ってくれるのかしら？

これも試す必要があります。


