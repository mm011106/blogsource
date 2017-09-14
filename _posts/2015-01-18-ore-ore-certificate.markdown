---
layout: post
title: "オレオレ証明書を作ってみようかと　準備"
date: 2015-01-18 11:28:32 +0900
comments: true
categories: [SSL, "CA certificate", mosquitto]
---

現在、セキュアな通信はSSHで確保する方針ですが、証明書ベースの暗号化についても考えておいてもいいのじゃないかなあとおもい、いわゆる「オレオレ証明書」を試してみようかと思った次第です。
ここでは、まず勉強。
<!-- more -->

英語のwebでは"self-signed certificate"なんて言われていますが、日本では「オレオレ詐欺」にちなんで、「オレオレ証明書」と呼ぶのがツウのやりかたのようです。でも、これは結構いいネーミングじゃないかと思います。日本で「オレオレ」というのはかなりダーティな印象があるので、「これ、オレオレ証明書だよね」というと、直感的に騙されそうな感じがします。

昔の記事（2007年頃）ですが、銀行のオンラインバンキングなどでもオレオレ証明の一種が使われていたということで、間違ったITリテラシを教えてしまうという危険が[指摘](http://takagi-hiromitsu.jp/diary/20071117.html)されています。

このページの終わりの方には証明書の分類が挙げられており、一口に「オレオレ」といってもいろいろあるのだよ。ということがわかります。

今回つくろうとしているのは、ここで言うところの「第四種オレオレ証明書」です。  
とくに悪意があってself-signedにするわけではなく、パブリックに公開されるサーバではないし、特定の用途にしか使わない、自分たち専用の暗号化経路ということです。

ポリシーとしては、

- 経路の暗号化には証明書ベースの共通鍵暗号化
- ログイン制御にはユーザ名/パスワード
- ポート番号の変更

という3本柱で考えてみたいと思います。

証明書ベースの認証を行うという事は、だれでも（ポート番号がわかれば）暗号化経路を作れる、ということになります。そのため、実際にブローカに接続できるかどうかは、ユーザ名による認証を行う必要があると思ったからです。実際には、IPやドメイン名の制限など、可能な接続制限を取る必要があると思います。

## mosquitto.conf と mosquitto_pub/subの暗号化についての記載

まずは、いま使っているクライアント・サーバアプリケーションが証明書ベースの暗号化にどのように対応しているかを確認しておきます。
まずは、ブローカ側の`mosquitto.conf`から

### mosquitto.conf
#### Authentication

>The authentication options described below allow a wide range of possibilities in conjunction with the listener options. This section aims to clarify the possibilities.

>以下に示す認証のオプションはリスナーのオプションとともに活用することで広いレンジの可能性を提供します。このセクションはその可能性を明らかにします。

>The simplest option is to have no authentication at all. This is the default if no other options are given. Unauthenticated encrypted support is provided by using the certificate based SSL/TLS based options cafile/capath, certfile and keyfile.

>最もシンプルなものは認証をしないようにすることです。これは他のオプションが与えられない限り、デフォルトの設定になります。認証なしの暗号化はcafile/capathオプションにより証明書ベースのSSL/TLSから提供されます。

>MQTT provides username/password authentication as part of the protocol. Use the password_file option to define the valid usernames and passwords. Be sure to use network encryption if you are using this option otherwise the username and password will be vulnerable to interception.

>MQTTはプロトコルの一部としてユーザ名／パスワードによる認証を提供しています。
`password_file`オプションを使うことで有効なユーザ名とパスワードを設定できます。このオプションを使うときは経路の暗号化を必ず実施してください。そうしないとユーザ名／パスワードは盗聴の対象となってしまいます。


>When using certificate based encryption there are two options that affect authentication. The first is require_certificate, which may be set to true or false. If false, the SSL/TLS component of the client will verify the server but there is no requirement for the client to provide anything for the server: authentication is limited to the MQTT built in username/password. If require_certificate is true, the client must provide a valid certificate in order to connect successfully. In this case, the second option, use_identity_as_username, becomes relevant. If set to true, the Common Name (CN) from the client certificate is used instead of the MQTT username for access control purposes. The password is not replaced because it is assumed that only authenticated clients have valid certificates. If use_identity_as_username is false, the client must authenticate as normal (if required by password_file) through the MQTT options.

>証明書ベースの暗号化を行う場合、2つのオプションが認証に影響を与えます。  
1つめは`require_certificate`で、trueもしくはfalseに設定できます。  
falseの場合、クライアントのSSL/TLSコンポーネントはサーバが正確であるかどうかを確かめます。しかし、クライアントは何もサーバに提供する必要はありません。認証はMQTTにビルトインされたユーザ名・パスワードに制限されます。  
trueの場合、クライアントは有効な証明書を接続するために提供する必要があります。  

>このケースでは2つ目のオプション、`use_identity_as_username`が関連してきます。  
これをtrueにセットするとアクセスコントロールのために証明書共通名(CN)がMQTTのユーザ名の代わりに使われます。パスワードは、認証取得をしたユーザのみが有効な証明書を持っていると考えているため変更されません。  
`use_identity_as_username`がfalseの場合、クライアントは通常と同じようにMQTTによって認証されます。（password_fileが必要です）

>When using pre-shared-key based encryption through the psk_hint and psk_file options, the client must provide a valid identity and key in order to connect to the broker before any MQTT communication takes place. If use_identity_as_username is true, the PSK identity is used instead of the MQTT username for access control purposes. If use_identity_as_username is false, the client may still authenticate using the MQTT username/password if using the password_file option.
（ここは事前共有キーの話なのでパス）

>Both certificate and PSK based encryption are configured on a per-listener basis.

>この証明書ベース、PSKベースの暗号化どちらの場合でもリスナー単位での設定になります。

>Authentication plugins can be created to replace the password_file and psk_file options (as well as the ACL options) with e.g. SQL based lookups.  

>たとえば、SQLデータベース参照などの認証プラグインを、ユーザ名／パスワード認証とPSKファイルのオプションをリプレイスするために設定することも可能です。

>It is possible to support multiple authentication schemes at once. A config could be created that had a listener for all of the different encryption options described above and hence a large number of ways of authenticating.

>また複数の認証スキームを一度にサポートするようにすることも可能です。設定ファイルは一つのリスナーに対して上記にあるすべての違った暗号化のオプション、つまりいくつもの認証方法を設定することも可能です。


と、ここまでの説明では、「いろんな方法で暗号化、認証ができます」というところだけしかわかりませんでした。  
これからやろうとしている証明書ベースの暗号化では、認証と絡めることもできるし、それとは別にすることもできる、というところでしょうか。

証明書をクライアントが提供するようなやり方もできるように読めます。
確かに、クライアントが証明書を提示するという方が理にかなっているかもしれません。今想定しているような構成では、接続してきたクライアントが「偽物」である可能性を排除したいわけで、一般的なHTTPプロトコルでの証明書のようにサーバ側の真贋をユーザのセキュリティのために提供するのとは全く逆になります。

兎にも角にも手を動かさないことには勉強できないので、まずは、証明書をサーバが提供するという、一般的なやり方の設定をしてみようと思います。証明書ベースの暗号化経路を確立、次にユーザ名／パスワードでクライアント認証をする、という手順になります。

今日のところは、これまで。


