---
layout: post
title: "Raspberry Pi 2　を試して見ました"
date: 2015-02-07 10:58:26 +0900
comments: true
categories: raspi
---
あまり流行りものに飛びつかないんですが、先週何気なくwebを眺めてたら、"Raspberry Pi 2　発売!"という記事があって、「どうせ買うことになるし、在庫あれば買ってみようかな」とおもいまして。

<!-- more -->
で、RS componentsに行ってみると「在庫あります」ということでしたので、何も考えず購入。ハッピーなことに金曜日到着しました。

今あるRaspiのSDカード(OS)で動くかな、とおもって試してみたらやっぱりダメで最新版OSをダウンロード。結局このダウンロードが3時間かかってしまい、動かしたのは土曜日の朝になりました。

やっぱり起動はキビキビしていますね。デバグの時の再起動などがやりやすくなります。それとスタートアップ画面左上の「ラズベリーアイコン」が4つになってます。4コアという事でしょうね。

で、気になる速度ですが、RSA鍵を作るopensslコマンドで試して見ました。

結果からいうと、このプロセス自体がシングルコアで動作するので、クロックアップ分の速度アップとなりました。

具体的には、50回　2048bitRSA鍵を作る下記のようなスクリプトを作りこれを更にシェルで10回やって、それぞれの実行時間を測るというものです。

```sh Script for testing computaiton time
$ cat keygen.sh 
#/bin/sh
for i in {1..50}
do 
   sudo openssl genrsa -rand ./rand.txt  2048 > key.txt
done

# このスクリプトを10回試行してそれぞれの処理時間を書き出します。

$ for j in {1..10} ; do (time -p ./keygen.sh ) 2>&1 | grep 'user' | awk '{print $2}' >> out_raspi2.txt ; done


```

鍵の乱数の種は同じ物を使用。もともとgenrsaの実行時間は乱数生成に左右されるので、あまり正確ではないかもしれませんが、平均値と分散の変化を見ればいいかなと。

結果

----

__Raspberry Pi 1:__

310.34,
299.83, 
285.74, 
261.49, 
342.31, 
353.19, 
340.26, 
318.30, 
367.67, 
322.97 

平均：320.21, 標準偏差：32.3

----

__Raspberry Pi 2:__

204.71,
208.46,
214.48,
197.13,
229.73,
222.12,
207.66,
226.37,
194.71,
244.09

平均：214.95, 標準偏差：15.55

----

平均が0.67倍、標準偏差が0.48倍でした。

クロックの増加分で0.78倍になるはずなので、それよりちょっと速いですね。

`top`でCPUの状態を見てみると。

```sh result of top command
$ top

top - 01:55:44 up 20 min,  3 users,  load average: 1.00, 1.00, 0.65
Tasks:  90 total,   2 running,  88 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:    754256 total,   142940 used,   611316 free,     9484 buffers
KiB Swap:   102396 total,        0 used,   102396 free,    96900 cached

  PID USER      PR  NI  VIRT  RES  SHR S  %CPU %MEM    TIME+  COMMAND           
 2607 root      20   0  4220 3192 2896 R 100.0  0.4   0:03.89 openssl           :
  :
```

となっていて、３コアは遊んでいる状態です。ヘビーなプロセスが動いていてもそれに引っ張られること無く、他のプロセスが動かせるようになります。動画の配信などには最適かもしれませんね。

メモリが増えているのでRAM-DISKのサイズ拡大に期待していましたが、案の定大きくなっています。

`df`で確認してみます。

```sh comparison of the size of RAM-DISK
# Raspberry Pi 1
tmpfs              76560   41932     34628  55% /run/shm

# Raspberry Pi 2
tmpfs             150840       0    150840   0% /run/shm
```

150MBもあります。
有効に使いたいです。


