---
layout: post
title: "init.dスクリプトを書いてみる"
date: 2015-02-23 21:20:50 +0900
comments: true
categories: Raspi, init.d
---

以前ハードウエアのスイッチでRaspberry Piをシャットダウンする時に、GPIOを監視するデーモンをシステム起動時に動かすことを試しました。
今回、MQTTのデータ経路を暗号化するためのポートフォワードを起動時にオープンすることを試します。
<!-- more -->

当初の設計では、MQTTでのパブリッシュの時にSSHのポートフォワードを実行するようにしていましたが、外部から操作を可能に出来るようなファンクションも入れておこうということになり、コマンド用のトピックを定義、そこへのメッセージを監視する常駐プロセスを起動する運びとなりました。

そのためデバイスとしては、常にブローカと暗号化経路を維持するのがリーズナブルという判断で、システム起動時に暗号化経路を作るスクリプトをinit.dで作成します。

##init.d用スクリプトを見てみる
いろいろとwebを検索しましたが、なかなか「これだ」という回答を得られないままでしたが、[ここで](https://gist.github.com/atr000/643783)わかりやすいスクリプトを見つけたので、これを元に動作を検証していきます。  
このスクリプトはautosshを起動時に動かすためのinit.d用スクリプトです。
autosshはsshの動作を監視して、止まったら起動させる仕事をします。そのため、設定それ自身はsshのものと同じ（というかsshにそのまま渡す）になります。  
このスクリプトでは、コマンドオプションで全ての指定をしていますが、
今回の様なポートフォワードではパラメタが多くなり、間違えやすくなるのでconfigファイルを作って指定してあげます。  
また、ポートフォワード用のユーザを作ってそのユーザでsshを起動します。

まず、INIT INFO部分について記述がなかったので足しておきます。
INIT INFOが無いと登録したときにいろいろと文句を言われますし、そもそもきちんとしたランレベルで実行されないので、やっておいた方が良いかなと。

完成したスクリプトを最後の方に書いておきます。この内容を順番みていきます。

まずはコメントに見えますが、重要なINIT INFO。

- Provides:
  - このスクリプトが提供する「ファシリティ名」を定義します。このスクリプトを必要とするその他のスクリプトに、このスクリプトの状態を判断できるように名前を付けておきます。  
他のinitdスクリプトでこのスクリプトの必要性をこの名前（ファシリティ名）で指定しておけば、お互いの必要性を考慮して正しい順序でデーモンが起動するようになります。とのことです。
- Required-Start:, Required-Stop:
  - このスクリプトの起動・停止に必要な「ファシリティ」を記載します。  
この例では、「メタファシリティ」という物が使われていて、具体的なスクリプトの名前じゃ無く「こういう状態」というような指示です。この定義については[debianのwiki](https://wiki.debian.org/LSBInitScripts)をご覧頂いた方が良いかと思います。
- Default-Start:, Default-Stop:
  - このスクリプトを起動する／停止するランレベルの条件を設定します。通常、OSが動作状態の時は2のようです。

次に定義されている変数です。

- TUNNEL変数
  - ssh用のconfigの中で設定されているポートフォワード設定のhost名を指定します。実際の接続はポートフォワード用のユーザで実行されるので、 configファイルはポートフォワード用のユーザの中におきます。具体的には`~/.ssh/config`になります。  
さらに、この設定ファイルの中で「パスワード無し接続」をするために鍵を指定する必要がありますが、この鍵も同じ場所に入れるようにします。  
鍵へのパスはフルパスで指定した方が無難かと思ったので、そうしてあります。  
- USER変数
  - ポートフォワード用のユーザ名を指定します。start-stop-daemonでユーザを指定してコマンドを起動するのに必要となります。
- DAEMON変数
  - 起動するautosshを指定します。コメントにもありますように、リンクとかじゃなく実態を指定する必要があるようです。ここを変更すれば他のコマンド・スクリプトも起動できると思いますが、PIDが出来るとか出来ないとかコマンドによって違うようなので、そこら辺は試しながらstart-stop-daemonのオプションを調整する必要があるようです。
- PIFLILE変数
  - pidを保存するファイルを指定します。stopするときpidが必要ですのでpidファイルには正しい値が入っている必要があります。ファイル名そのものは`basename $0`(このスクリプト自身のパスを含まないファイル名)になります。スクリプト名に拡張子が入っていると、拡張子を含んだ形のpidファイル名になってしまいます。（例えばhogehoge.sh.pid）
- SCRIPTNAME変数
  - usageを表示するときに使います。daemonの起動に直接関係ありません。
- DESC変数
  - 起動スクリプト実行時に何が起動されているのか、を表示しますが、そのときのメッセージです。短めが（4wordぐらい）が良いかもしれません。daemonの起動に直接関係ありません。
- ASOPT変数
  - 起動するコマンドにオプションを与えるための変数です。今回の例では`-M 0 -N host名指定`だけです。

これらの変数を適宜設定すればokかと思います。


## ポイントは。。。

実は、autosshの起動の前にssh単体での起動を試しました。そのときの問題はpidの取得で、起動したsshのpidを上手く取得できませんでした。色々調べてみると、実行をバックグラウンドに移行するときの指定にコツが必要でした。

- ssh にもバックグラウンドに移行するオプション(-f)があるが、それを使うと上手くpidを取得できない
- そのため、start-stop-daemonコマンドの「起動したプロセスをバックグラウンドに移行する」オプションである`--background`指定する
- さらに、このオプションを指定した場合 `--make-pidfile`を付ける。

これらをきちんとやると、PIDファイルが出来て上手く行きました。ですので、これをそのままautosshに適用しました。

結果的に、コマンドラインは

```
start-stop-daemon --start --quiet --background \
--chuid $USER --user $USER --pidfile $PIDFILE  \
--make-pidfile --exec $DAEMON -- $ASOPT
```

となりました。他のコマンド／スクリプトのデーモン化では`--make-pidfile` `--background`は不要かもしれません。

- `--chuid` はdaemonを起動するときのユーザ名を指定します。
- `--user` はプロセスをチェック（起動しているかどうか）の時のユーザ指定のようです。

シンプルなスクリプトになりましたので、他の用途にも使っていこうと思います。

これを起動時に動かすためには、

`
sudo update-rc.d SCRIPTNAME defaults
`

としましょう。これ以前に周到に動作チェックをしたほうがいいですけど。


```sh mqtt-pf

#! /bin/sh
### BEGIN INIT INFO
# Provides:          mqtt-pf
# Required-Start:    $syslog $network sshd
# Required-Stop:     $syslog $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Port forward for MQTT protocol
### END INIT INFO

#
# Author:	Andreas Olsson <andreas@arrakis.se>
# Version:	@(#)autossh_tunnel.foo  0.1  27-Aug-2008  andreas@arrakis.se
# modified : 13-Feb-2015 mqtt.and@gmail.com #     
#
# For each tunnel; make a uniquely named copy of this template.

## SETTINGS

#
# specify a host name descibed in /home/${USER}/.ssh/ssh_config
TUNNEL="MY_Broker"

#
#  user name for port forwading
USER="pfuser"

#
# You must use the real autossh binary, not a wrapper.
DAEMON=/usr/lib/autossh/autossh
#
## END SETTINGS

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

NAME=`basename $0`
# NAME is always including the extension of $0
# the script should be named without extension for good looking
PIDFILE=/var/run/${NAME}.pid
SCRIPTNAME=/etc/init.d/${NAME}
DESC="SSH Tunnel for MQTT protocol"

# exit when test result = false
test -x $DAEMON || exit 0

export MQTT_PF_PIDFILE=${PIDFILE}
ASOPT=" -M 0 -N "${TUNNEL}

#	Function that starts the daemon/service.
#
#  ssh command is not able to make a pid file with -f (force background) option.
#  To obtain pid file properly, put --background, --make-pidfile option on the start-stop-deamon command
#    and force forground to ssh command.

d_start() {
	start-stop-daemon --start --quiet --chuid $USER --user $USER \
	    --background --pidfile $PIDFILE --make-pidfile \
	    --exec $DAEMON -- $ASOPT
	if [ $? -gt 0 ]; then
	    echo -n " not started (or already running)"
	else
	    sleep 1
	    start-stop-daemon --stop --quiet --pidfile $PIDFILE \
		--test --exec $DAEMON > /dev/null || echo -n " not started"
	fi

}

#	Function that stops the daemon/service.
d_stop() {
	start-stop-daemon --stop --quiet --pidfile $PIDFILE \
		--exec $DAEMON \
		|| echo -n " not running"
}


case "$1" in
  start)
	echo -n "Starting $DESC: $NAME"
	d_start
	echo "."
	;;
  stop)
	echo -n "Stopping $DESC: $NAME"
	d_stop
	echo "."
	;;

  restart)
	echo -n "Restarting $DESC: $NAME"
	d_stop
	sleep 1
	d_start
	echo "."
	;;
  *)
	echo "Usage: $SCRIPTNAME {start|stop|restart}" >&2
	exit 3
	;;
esac

exit 0

```


