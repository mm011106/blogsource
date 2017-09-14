---
layout: post
title: "起動スクリプトで環境変数を読み込む"
date: 2015-02-25 21:03:27 +0900
comments: true
categories: init.d, environment
---

環境変数でデバイスのパラメタを指定することを考えてみました。  
システムワイドの環境変数設定は`/etc/environment`に入れるようですが、実際にこれを使いたいと思うのは起動時、という場合が多いのでちょっと困りものです。

<!-- more -->

というのも、起動時の環境変数はユーザ環境での環境変数とは違う設定になっているからです。
試してみましたが、やはり/etc/environmentでの設定は起動時には有効になっていません。なので、必要なスクリプトでこれを読み込むことを考えてみました。

最初、

``` sh
cat /etc/environment | while read -r env; do export "$env"; done
```
とやってみたのですが、全くだめ。  

> __2014/2/26__ 追記  
> これはパイプじゃダメで、リダイレクトがいいみたいです。
> `while read -r env; do export "$env"; done < /etc/environment `  
> とするとうまく行きました。


環境変数がプロセスの中だけで有効なのかなと思い、ループの中で環境変数を表示してみてもだめ。  
その後、いろいろ検索してみると

```sh
for line in $( cat /etc/environment ) 
  do
    export $line
  done
```

というスクリプトを見つけました。

こちらを試してみました。このシェルスクリプト内のループの後で環境変数を表示させてみるとうまく設定できています。  
ただ、このシェルスクリプトをコマンドラインから実行して、帰ってくると設定した環境変数は無くなっています。

スクリプト内では環境変数を維持できそうなので、早速init.dの起動スクリプトで試してみました。

結果は、良好！。無事、設定された環境変数を読み出すことが出来ました。  

さらにもう一つ、大事なことを勉強しました。環境変数の設定は、

``` sh
VAR=hogehoge
# これが正しい
VAR="hogehoge"
# とすると、""も変数に入ってしまう
```

結果、init.dの起動スクリプトでは、必要最低限のPATH指定もやってあげてから、/etc/environmentの内容を読み込むようにしました。  
環境変数の読み出し時に、空行とコメントを取り外すためのgrepコマンドを入れています。

``` sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

#
#  Read environment 
#
for line in $( cat /etc/environment | grep -v '^\s*#' | grep -v '^\s*$') 
  do
    export $line
  done

```

また一歩進みました。  
これで、init.d用のスクリプトの中でも不自由することが少なくなるように思います。

それにしても、本題と関係ないところで結構時間をとってるなあ。。。。。


