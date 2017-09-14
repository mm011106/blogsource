---
layout: post
title: "mbedでedge deviceを作ってみる（という予定）"
date: 2016-01-23 19:43:05 +0900
comments: true
categories: mbed 
---


RaspberryPiでエッジデバイスを作って、実際にデプロイしてみました。今のところ順調にデータを送ってくれています。しかしながら、やっぱりlinuxはちょっと重たい感じもするし、値段はともかくオーバースペックという感もあります。

以前予定していた通り、mbedでの開発を考えてみました。これからならmbedOSベースだろうということで、とりあえずはmbedOSに慣れてみよう、ということで。。。。

<!-- more -->

一昨年の11月にリリースされると聞いていたので楽しみにしていましたが、現在[TechnologyPreview](https://www.mbed.com/en/development/software/mbed-os/releases/)のようです。

## mbedOSを導入
mbedのwebにある[Installation](https://docs.mbed.com/docs/getting-started-mbed-os/en/latest/installation/)を見ながら進めます。ホストはMacOSです。  
ここでは、直接インストールするのではなく、Dockerを使ってみます。  
私の理解では、Dockerは実行環境を仮想化する仕組みで、実際の計算機の実行環境と全く別に設定された「コンテナ」とよばれる環境中で作成したアプリケーションを実行できるものです。こうすることで、特定のアプリケーションを動かすための実行環境を使っている開発環境と別に用意できるため、開発環境上でのソフトウエアのコンフリクトを防げるとともに、「コンテナ」をそのまま実際の実行環境に移行できるので「開発時の環境と実行時の環境を全く同じ」にできます。

### 1. dockerを導入
[docker](https://www.docker.com/docker-toolbox)のwebからMac版をダウンロードしてインストール。  
インストール途中で「起動してみてもいいよ」的な状態になるので、とりあえず無視して終了。

### 2. docker 起動
Launchpadから「Docker Quickstart Terminal」を起動すると、クジラのアスキーアートとともに起動。  
何がなんだか解らないので、tutorialにある
```
$ docker run hello-world
```

を実行してみると、

```

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/

```
ということで、新しい「コンテナ」にhello-worldのイメージを展開してその中にある実行ファイルを実行して、それがDocker clientにこのメッセージを送信、それがターミナルに表示されている、という形になっているとのこと。

インストールはうまく行っているということなので、次に。

### 3. yottaのインストール
mbedOSのアプリケーションはすべてyottaによって作成される必要がある、ということですので、[こちらを参考に](https://docs.mbed.com/docs/getting-started-mbed-os/en/latest/docker_install/)yottaをインストールします。

`$ docker pull mbed/yotta`

これでdockerのmbed/yottaコンテナを持ってきます。yottaのコマンドをこのコンテナに対して発行することで独立した環境でyottaを実行できる、という感じかとおもいます。  
そのためのスクリプトとして[yotta.sh](https://raw.githubusercontent.com/ARMmbed/GettingStartedmbedOS/master/Docs/Scripts/yotta.sh)を使うようです。
これをwgetして、

`$ ln -s {pathToTheDownloaded}/yotta.sh /usr/local/bin/yotta`

とすることで、`$ yotta ~ ~ ~` の形でyottaをコンテナ内で実行できるようになります。

## 試してみる
[blinky](https://docs.mbed.com/docs/getting-started-mbed-os/en/latest/FirstProjectmbedOS/)を試してみます。といってもmbedOSでサポートされているボードが手元に無いので、途中までですが。。。。

先のリンクにあった通りに実行してみます。  

```
$ mkdir blinky
$ cd blinky
$ yotta init
```
とすることで、ディレクトリが作業用としてイニシャライズされます。  
モジュール（作ろうとしているアプリケーション）の名前やバージョン、ライブラリなのか実行形式なのか、短い説明、ライセンスを聞かれますので、適宜答えると終了です。

次に、ターゲット環境を設定。
現状、おなじみのmbedデバイスはあまりサポートされていないようです。  
次のコマンドでターゲットの設定ファイルがリストアップされます。

`$ yotta search target`

といっても、あまり見通しのよいリストではない感じですね。
ここでは、サンプル通り

`$ yotta target frdm-k64f-gcc`

としてみます。mbedアカウントのログインが必要のようです。コンソールに出てくるURLをコピーしてブラウザでログインすると処理が進みます。

しばらくして、ダウンロードとデプロイが完了したあとに、チェックしてみます。

```
$ yotta target
frdm-k64f-gcc 2.0.0
kinetis-k64-gcc 2.0.0
mbed-gcc 1.1.0
```

なにやら設定されたようです。

次にdependenciesをインストールします。ライブラリ、ということでしょうか。

```
$ yotta install mbed-drivers
info: get versions for mbed-drivers
info: download mbed-drivers@0.11.8 from the public module registry
info: dependency mbed-drivers: ~0.11.8 written to module.json
info: get versions for mbed-hal
info: download mbed-hal@1.2.1 from the public module registry
info: get versions for cmsis-core
info: download cmsis-core@1.1.1 from the public module registry
info: get versions for ualloc
info: download ualloc@1.0.3 from the public module registry
....
....
```

bliknkyのソースコードを用意して、`./source/app.cpp`として保存します。

```cpp
#include "mbed-drivers/mbed.h"

static void blinky(void) {
    static DigitalOut led(LED1);
    led = !led;
    printf("LED = %d \r\n",led.read());
}

void app_start(int, char**) {
    minar::Scheduler::postCallback(blinky).period(minar::milliseconds(500));
}
```

ビルドしてみます。結構時間がかかります。２分とか。（最初だけかも）

```
$ yotta build
info: generate for target: frdm-k64f-gcc 2.0.0 at /yotta_home/work/bliknky/yotta_targets/frdm-k64f-gcc
GCC version is: 4.9.3
-- The ASM compiler identification is GNU
-- Found assembler: /usr/bin/arm-none-eabi-gcc
-- Configuring done
-- Generating done
-- Build files have been written to: /yotta_home/work/bliknky/build/frdm-k64f-gcc
[130/130] Linking CXX executable source/bliknky

```

てな感じで終了。

`./build/frdm-k64f-gcc/source/`に`blinky.bin`というファイルが出来ていて、それがターゲットの実行ファイル（の転送できる形）のようです。

手元にボードが無いのでここまで。

本筋ではないですが、dockerはかなり便利ですね。インストール作業無し（コンテナを引っ張ってくるだけ）でこれだけのツールがReadyToUseになります。コンテナを作ってくれた人には感謝です。  
大抵は、インストールがうまく行かなくて２、３度はコケてしまうこともあり、場合によってはついにあきらめてしまうこともあるので。。。。


