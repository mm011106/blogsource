---
layout: post
title: "autosshが起動スクリプトでうまく動かない訳"
date: 2015-02-28 14:10:43 +0900
comments: true
categories: autossh, init.d
---

エラーする原因をつかみました。
初めてのホストに接続するとき良く目にする以下のメッセージ。
<!-- more -->

```
The authenticity of host 'mybroker.ABC_corp.jp (xxx.xxx.xxx.xxx)' can't be established.
ECDSA key fingerprint is xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx.
Are you sure you want to continue connecting (yes/no)? 
```


というのが、曲者でした。  
接続しようとしているユーザの~/.ssh/known_hostsに接続先の「ホストキー」がないと「本当に接続していいのか？」と聞かれます。  
これは、ssh で接続するときに、先方のホストが本当に正しい（自分の知っている）物かどうかを確認するために、自分が持っている接続先の鍵と送られてきた鍵が等しいかどうかを確認しますが、その結果「違う」と判断された場合出るようです。なので通常、そのホストに初めて接続する場合に出てきます。

インタラクティブにやっているときはyesでいいのですが、スクリプトで実行されているときにこのような質問が来るとスクリプトが止まってしまいます。
本来的には、きちんとKnown_hostsに接続先を登録するべきだと思いますが、どうも「ちゃんとやってるのに聞かれる」という場合があるように見て取れます。私の設定が悪いんでしょうけれど。。。

ちなみに、ホストキーは/etc/ssh/ssh_host_ecdsa_key.pubです。

今回は、このメッセージが出ないように設定することで回避します。

基本的には、known_hostsに接続しようとするホストの鍵があれば良いので、一度手動で接続するというのも一つの手です。ただ、自分の場合、hostkeyがあるはずなのにうまく行かない、という事があります。

具体的には、StrictHostKeyChecking をno に設定します。ですが、/etc/ssh/ssh_config で設定変更してしまうと全ユーザが影響を受けてしまうので、ローカルの設定を変更します。

~/.ssh/configを開いて、最初の方に `StrictHostKeyChecking no`　を追加します。

これでok。

configを使ったautosshもこれで安定したような気がします。



