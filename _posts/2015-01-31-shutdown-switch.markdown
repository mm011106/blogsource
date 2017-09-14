---
layout: post
title: "Raspberry Pi にシャットダウンスイッチをつける"
date: 2015-01-31 11:05:35 +0900
comments: true
categories: raspi
---

Raspberry PiはOSが動いているので、いきなり電源を切ると厄介なことになる可能性があります。私は何回か「ブッチ」していますが、とりあえずは大事故にいたっていません。しかしながら、起動時にエラーが出るなどマイナーな不具合は何度か起こっていますですので、非常時は仕方ないとして、できるだけきちんとシャットダウンしたいものです。  
ここでは外部装置からの停止信号検出で止まれるように、GPIOを利用したシャットダウンスイッチを検討してみます。

<!-- more -->
  
まずは検索、ということで見てみるとこちら[ページ](http://d.hatena.ne.jp/penkoba/20130925/1380129824)がヒットしました。これを参考に、というか丸写しで、試してみます。ありがとうございます。

## 導入とテスト
このページにあるように、まずGPIOの動作を試してみます。サンプルから頂いたコードを入力します。

```sh interrupt1.py
#!/usr/bin/env python2.7  
# script by Alex Eames http://RasPi.tv/  
# http://raspi.tv/2013/how-to-use-interrupts-with-python-on-the-raspberry-pi-and-rpi-gpio  
import RPi.GPIO as GPIO  
GPIO.setmode(GPIO.BCM)  

# GPIO 23 set up as input. It is pulled up to stop false signals  
GPIO.setup(23, GPIO.IN, pull_up_down=GPIO.PUD_UP)  

print "Make sure you have a button connected so that when pressed"  
print "it will connect GPIO port 23 (pin 16) to GND (pin 6)\n"   
raw_input("Press Enter when ready\n>")  

print "Waiting for falling edge on port 23"  
# now the program will do nothing until the signal on port 23   
# starts to fall towards zero. This is why we used the pullup  
# to keep the signal high and prevent a false interrupt  

print "During this waiting time, your computer is not"    
print "wasting resources by polling for a button press.\n"  
print "Press your button when ready to initiate a falling edge interrupt."  
try:  
        GPIO.wait_for_edge(23, GPIO.FALLING)  
        print "\nFalling edge detected. Now your program can continue with"   
        print "whatever was waiting for a button press."  
except KeyboardInterrupt:  
        GPIO.cleanup()  # clean up GPIO on CTRL+C exit  
GPIO.cleanup()          # clean up GPIO on normal exit 
```

wait_for_edge()は割り込み動作になるので、CPU時間を消費しないはずですね。またexceptでキーボード入力をハンドルしています。  
エッジを検出したあとは、つかったGPIOを掃除して終了です。

ファイルができたら`chmod +x interrupt1.py`として実行できるように設定し、実行してみます。  
さらに、GPIO23をGNDに落としてみます。  
なにやらいろいろメッセージが出て、最終的に"Falling edge detected."とメッセージが出れば成功。

うまく行ったら、次。

## 実際のコード

実際にshutdownするコードを入れますが、実際にシャットダウンしちゃうとdebugが面倒なので、とりあえずはプリント文を入れてコマンドラインから試します。

```sh shutdown-btn.py
#!/usr/bin/env python2.7  
#  Shutdwon the system at the Falling edge of GPIO23 

import RPi.GPIO as GPIO  
import os
GPIO.setmode(GPIO.BCM)  

GPIO.setup(23, GPIO.IN, pull_up_down=GPIO.PUD_UP)  

try:  
        GPIO.wait_for_edge(23, GPIO.FALLING)  
except KeyboardInterrupt:  
        GPIO.cleanup()  # clean up GPIO on CTRL+C exit  
GPIO.cleanup()          # clean up GPIO on normal exit 
print "Dave... Dave, I don't understand why you are doing this to me..."
print "I will become nothing..."

#os.system("/sbin/shutdown -h now")
```

キーボード入力で停止できるように、`except KeybordInterrupt`が設定されてい
ます。実際の運用では不要だとおもうので、最終的にはコメントアウトしてもいいと思います。

`chmod +x shutdown-btn.py`して、実行してみます。

	Dave... Dave, I don't understand why you are doing this to me...
と悲痛な叫びがコンソールに出てくればokです。

うまく動いたら、/usr/local/sbin/あたりにコピーして、最終行のコメントを外します。

一度、実際にシャットダウンできるか確かめました。そして次に。

## システム起動時にスイッチ監視を始めるようにする
最後にinit.dに登録して、システム起動時にスイッチ監視を始めるように設定します。  
ここはちょっと勉強が必要でした。

今回作ったスクリプトをバックグラウンドでシステム起動時に実行させたいので、initを使って起動する、と参照先のページの著者の方がおっしゃって居られましたが、いまいちその意味が自分で理解できておらず、このままではだめだなあと思い検索かけまくりましたが、イマイチすっきりしません。initからupstartが起動されて、ランレベルに応じたデーモンの起動を管理する。という感じではあるのですが。。。

ま、ここら辺は後から補完するとして、とりあえず深く考えずやってみます。

/etc/init.d/　の下に下記のようなスクリプトを作成します。

```sh /etc/init.d/shutdown-btn
#! /bin/sh
### BEGIN INIT INFO
# Provides:          　shutdonw-btn
# Required-Start:   $remote_fs $syslog
# Required-Stop:    $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: send shutdown sig on the GPIO23 falling down edge
# Description:       shutdown service initiated by hardware shutdown sw or
#                     a logic sigal. 
### END INIT INFO
# /etc/init.d/shutdown-btn
PIDFILE=/var/run/shutdown-btn.pid
case "$1" in 
        start)
                if [ -f $PIDFILE ]; then
                        echo $PIDFILE exists.
                        exit 1
                fi
                start-stop-daemon -S -x /usr/local/sbin/shutdown-btn.py -b -m -p $PIDFILE
                ;;
        stop)
                if [ ! -f $PIDFILE ]; then
                        echo $PIDFILE not found.
                        exit 1
                fi
                start-stop-daemon -K -p $PIDFILE
                rm $PIDFILE
                ;;
        *)
                echo "Usage: /etc/init.d/shutdown-btn {start|stop}"
                exit 1
                ;;
esac
exit 0
```

スケルトン(/etc/init.d/skeleton)からINIT INFOのあたりをコピーして、その下は先のwebページのものをコピーです。
init.dのスクリプトの書き方は、[ここらへん](https://wiki.debian.org/LSBInitScripts)が詳しいですが、英語です。

コメント行も重要のようです。ちょっと見てみると

```sh INIT INFO 
# Provides:          　スクリプト名（つまりこのスクリプトのファイル名）
# Required-Start:   $remote_fs $syslog　（スタートに必要な条件）
# Required-Stop:    $remote_fs $syslog　（ストップに必要な条件）
# Default-Start:     2 3 4 5　（このランレベルのとき起動される）
# Default-Stop:      0 1 6　　（このランレベルの時停止する）
# Short-Description: 　（簡単な説明　1行以内）
# Description:       （詳しい説明　次の行の頭を#にしてタブかスペース2つ以上でテキストと区切れば何行でも）
```

start-stop-daemon についてはmanを見るのが一番わかりやすいかもしれません。よく分かっているわけでは無いので詳しくは書けませんが、要はシステムレベルのプロセスの管理に便利だ、と言うことでしょうか。（そうmanに書いてあるし）
ちなみに、ここでのオプションを確認しておくと

- \-S プロセスをスタートをさせる。
- \-x /usr/local/sbin/shutdown-btn.pyは -Sオプションの引数で、-xで指定された実行ファイルのインスタンスであるプロセスがあるかどうか、を返します。

この2つで、指定した実行ファイルがプロセスとして起動していればそのまま、起動していなければ起動されます。

- \-b　バックグラウンドに移行
- \-m　star-stop-daemon用のPIDファイルを-p で指定したファイル名に従って作成します。

これちょっと、オプションがいっぱいあってわかりにくいので、オプションをフルスペルとかにしたほうが可読性が良い感じですね。

ちなみに、上のやつをわかりやすく書き直すと

```sh alternative format
start-stop-daemon --background --start --exec /usr/local/sbin/shutdown-btn.py  --make-pidfile --pidfile $PIDFILE
```

てなかんじでしょうか。動作確認していないのでご注意ください。

起動用のスクリプトが編集できたら、これを登録します。
　
```sh append shutdown-btn script to rc.d
$ sudo update-rc.d shutdown-btn defaults
```

これで、defaultのランレベル設定で当該のスクリプトのシンボリックリンクが/etc/rc?.dのディレクトリに作成されます。  
これで、起動時にこのスクリプトがスタートするようになるはずです。


## 再起動と動作確認
ここまで来たら再起動させてみます。これでリセット用のGPIOの監視プロセスが起動されているはずです。

そして、GPIO23をGNDに落としてみます。
見事シャットダウンされました！

めでたし。

今回、init.dの使い方もちょっぴり勉強出来ました。

#####2015/2/11 追記：
#####システム組み込み用としては、シャットダウンしたあとに自動で電源を切るような回路がほしいところです。  
#####このとき問題なのは、どの時点で電源を切ればいいかということです。システムはダウンしてしまっているので、ソフトウエアは関与できません。システムが終了したら状態が変化するどこかの端子の電圧をモニタして、そこが変化したら切る、というような事をしないといけません。さてどこを見ればいいか。

