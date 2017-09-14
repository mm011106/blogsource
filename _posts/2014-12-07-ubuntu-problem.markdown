---
layout: post
title: "ubuntuのランチャとかが消えてしまう件"
date: 2014-12-07 13:19:44 +0900
comments: true
categories: [ubuntu]
---

apt-get upgradeとかコマンドラインから実行するとubuntuのランチャなどが消えてしまうという状態に時々なります。ランチャだけならいいのですが、メニューが表示されるところまで消えてしまうし、文字入力もできなくなります（つまり、見えはするけど手が出せない状態）
<!-- more -->

検索してみたところ、[この](http://m12i.hatenablog.com/entry/2014/05/24/100539)ページにあるような症状にぴったりだったので、書かれている復活手順を踏んでみました。以下はその引用です。　**　_mizukyfさん、ありがとうございます。_ **

- ログイン後 Ctrl＋Alt＋F1キー入力 でコンソール画面に遷移。

- 「（マシン名） login:_」という表示されるのでログイン。

- 「DISPLAY=:0.0 gnome-terminal」と入力しEnter。gnomeの上のターミナルを起動

- Alt＋F7キーを押下するとデスクトップ画面に復帰するが端末ウィンドウが新たに表示されている。

- このターミナル上で「dconf reset -f /org/compiz/」と入力してEnter。

- 「sudo shutdown -r now」と入力してEnter。OSが再起動する。

多分、`dconf reset -f /org/compiz/`が重要なので、コンソールでログインしたあとにこれを実行すればいい、という気もします。
今度不具合になったら試してみましょう。

PS:  
[compiz](http://ja.wikipedia.org/wiki/Compiz)を調べてみたところ、ウインドーマネージャなのですねえ。  
[dconf](http://en.wikipedia.org/wiki/Dconf)はいろいろな設定を一気に引き受けるユーティリティ。あるwebの解説によれば「Windowsのレジストリシステムのようなもの」とのこと。




