---
layout: post
title: "IBM IoT foundationを試してみた"
date: 2015-03-29 09:42:17 +0900
comments: true
categories: mqtt, cloud 
---
デバイスの方で手一杯で、MQTTブローカをVPSで立ち上げるとか、データのビジュアライズとか解析とかまで手が出ないので、SaaSというかIaaSをためしに使って見ることにしました。
<!-- more -->

最近では、wolfram もIoT向けのサービスを提供しているぐらい、いろんな会社でやっていますが、私はMQTTを使っていることもあり、本家IBMの"IBM Internet of Things foundation"を試し始めました。  
イマイチわかってはいないのですが、IBM Bluemixというサービスがあって、その拡張としてMQTTを扱えるようにした、というような感じだと認識しています。

## まずはアカウントを作る

で、早速[IBM IoT foundation](https://internetofthings.ibmcloud.com/)のページに行って登録しました。
ページの上の方にSignUp、下の方に Try Quick startのボタンがあったので、QuickStartの方から登録しました。支払い条件とかを聞かれませんでしたので、完全にお試しのようです。
ページの上の方にあるSignUpではコースを選んだりするところがあり、こちらは「購入」という感じです。ただ、デバイスが20台以下、月間のデータ転送量が100MB以下、月間の使用ストレージが1GB以下だと無料のようです。

まずは、アカウントを作ります。
多分[ここから](https://apps.admin.ibmcloud.com/manage/trial/iot.html)入れば大丈夫だと思います。。。。大手のIT企業のwebページはどうも複雑で、昨日やったことをトレースしようと思ってもうまく行きませんでした。

登録すると、confirmation e-mailが届きますので指定されたURLをクリックして完了です。
このとき表示されるwebページが「ダッシュボード」となります。左上に”Organization: xxxxx (Trial)"のように表示されていると思います。この6文字のコードがデバイスを接続するホスト名につくIDの"organaization Code"となりますので、メモしておいてください。

##デバイスを登録

次にやることは、デバイスの登録です。
このページの左側コラムにあるように、「ページのdeviceタブを押して」「Add Deviceを選択」します。  
そうすると、新しいページに移動します。ページ中央に"Device Type"を入力する欄、その下に"Device ID"を入力する欄があります。これはデバイスを識別するためのもので、自分で勝手に決めていいようです。私はDevice Type に製品の型式、Device IDに、指示のようにMACアドレス（コロン抜きで）を入れました。

入力を終えると、次のページでデバイスをクラウドに接続するための大事な情報が表示されるので、メモしておきます。

```sh DEVICE IDs
org=asdfjk
type=AD
id=abcd12345654
auth-method=token
auth-token=(@dS&34+#2upCxP1
```
のような感じです。

##MQTTで接続

さあ、これで接続だあ！とmosquitto_pubでデータをパブリッシュしてみます。

ひとまず、ホームとなる「ダッシュボード」に戻ります。URLは`https://internetofthings.ibmcloud.com/dashboard/#/organizations/${org}/home`のようになるはずです。迷ったらここに戻りましょう（私は何度と無く迷子になりました）。 
${org}には先の「大切な情報」の中のorgを入れます。

Deviceタブをクリックして、先ほど登録したデバイスが出てくるのを確認します。

次に、ページの一番上、グレーのバンドのところに"QuickStart"というところがあるので、それを押してみます。すると、新しいページに移動して、左側にMACアドレスを入力する欄が出てきます。なんだかわからないまま、うながされるまま登録したデバイスのMACを入れてみます。

すると、「メッセージを待ってます」みたいなセリフが出て「Device IDは登録されていますが、データが来ていません。有効なデータが来ればビジュアライズしますよ」というメッセージが見えます。

ならば、web上でビジュアライズしてもらおうじゃあないか、ということでMQTTでパブリッシュしてみます。

とはいうものの、何をどうしていいやら。。。。。webを捜し回ること1時間ほど。[Documentation](https://docs.internetofthings.ibmcloud.com/messaging/devices.html)
を発見。結局これが一番わかり易い感じです。

これによれば、MQTTのパラメタを以下のように指定すればつながるようです。
shellスクリプト風の表記をして見ました。

- client_id : d:${org_id}:${device_type}:${device_id}
- username : "use-token-auth"
- password : ${auth-token}

必要な情報はすべてデバイスを登録した時点で「重要な情報」として出てきたものです。

同時にトピックツリーも重要ですが、デフォルトでこのような構成になっているようです。

- iot-2/evt/${event_id}/fmt/${format_string}

event_id,format_stringは任意に（適当に）決めていいようです。ただ、クラウドで処理する分には決まりがあるようです。（ここらへん未だに解明できていない。。。）

具体的にmosquitto_pubのコマンドパラメータに展開すると、

```sh 
mosquitto_pub -i d:asdfjk:AD -h asdfjk.messaging.internetofthings.ibmcloud.com \
-u use-token-auth -P "(@dS&34+#2upCxP1" \
-t iot-2/evt/hogehoge/fmt/json -m "hello IBM"

```

のような感じになるはずです。

で、早速、適当なデータをPublishしてみるも[web上](https://quickstart.internetofthings.ibmcloud.com/#/device//sensor/)には何も現れません。

また、ダッシュボード上のデバイス一覧を見ると、データが受信されていることはわかったので、一応つながっているのだな。ということはわかりました。  
webでのデータ表示方法の解明まで時間がかかりそうだったので、データが来ていることが確認できた所でやめておきます。

##ダメなら、ブローカとして使えるか？

デバイスが送ったデータをクラウド上で表示するのが定番でしょうけれど、どうもうまく行かなかったので、クラウドをふつうのブローカと同じように使えないかとやって見ました。   
結論から言うと、ストレートフォワードなブローカから比べると面倒くさいけど、できました。

```sh
# デバイス側
mosquitto_pub -i d:${ORG}:${TYPE}:${ID} \
 -h AAAAAA.messaging.internetofthings.ibmcloud.com \
 -u use-token-auth -P "${auth-token}" \
 -t iot-2/evt/hogehoge/fmt/text -m "hello IBM"

# サブスクライブ
mosquitto_sub -i a:${ORG}:${TYPE} \
 -h AAAAA.messaging.internetofthings.ibmcloud.com \
 -u "${API-KEY}" -P "${API-AUTH-Token}" \
 -t iot-2/type/${TYPE}/id/${ID}/evt/hogehoge/fmt/text

```

サブスクライブするためにはAPI-KEYというものが必要になります。これを発行するためにダッシュボードに戻ります。

ダッシュボードのページに"API KEYS"というタブがあるので、それをクリックします。さらに"New API Key"というハイライトがあると思うので、それをクリックします。

ポップアップが現れて、Key, Auth Tokenが表示されますので、メモしておきます。サブスクライブするときにこのキーを指定する必要があります。

##まとめ

最初に"Organaization"を作ります。IoT foundationに登録すれば自動的に自分のOrganaizationコードが発行されます。

次に、デバイスを登録します。登録すると接続のためのTYPE,ID,Tokenが発行されます。

さらに、アプリケーション（デバイスのデータを受信・利用する側）のIDを登録します。登録すると key, Auth-tokenが発行されます。

デバイス側のトピックツリーとアプリケーション側のトピックツリーはちょっと違っています。デバイス側のツリーはアプリケーション側のツリーのサブセットという感じです。アプリケーションは多くのデバイスからのデータを取り込む必要があるためそうなっているのかと。

デバイス、アプリケーションkeyは「ダッシュボード」"https://internetofthings.ibmcloud.com/dashboard/#/organizations/${org}/home"からいつでも確認できます。
デバイスからのパブリッシュのタイムスタンプも確認できるので、動作状況を軽くチェックするにはいいかもしれません。

とりあえず、IBM IoT foundationをMQTTブローカとして使えることがわかりました。目的であるwebアプリケーションを作るところまではまだまだですが、ドキュメントをよく読めばある程度わかるかな、という感じです。

