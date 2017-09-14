---
layout: post
title: "オレオレ証明書を作ってみようかと　実践１"
date: 2015-01-31 20:29:12 +0900
comments: true
categories: [nginx,TLS,SSL]
---

先に投稿した、オレオレ証明書の続きです。

今回は、実際の証明書を作りwebサーバに導入して動作確認までやって見ました。

<!-- more -->

## オレオレ証明書を作る
いったいどこから手を付ければ良いか分からないので、検索してみます。
いろいろなところで例が示されていますが、opensslで作れそうな感じです。とくに[こちらのページ](http://d.hatena.ne.jp/ozuma/20130511/1368284304)に非常にわかりやすくまとめられていましたので、こちらをそのまま順番に試していこうと思います。

詳しい解説は、そちらをご覧ください。

### 1.秘密鍵を作る
まずは、秘密鍵(server.key)を作ります。

```sh 
$ openssl genrsa 2048 > server.key
```
これで2048bitのRSAキーが作られます。参考ページの解説によると、この情報の中に暗号化に必要な全てのものが入っています。ここから公開鍵も作られます。

中身を見てみます。

```sh 
$ openssl rsa -text < server.key 
Private-Key: (2048 bit)
modulus:
  :
  :
```
見てもよく分かりませんが、なんか出来てます。

### 2.証明書署名リクエストファイルを作る

次に、証明書にサインしてもらうためのリクエストファイルを作ります。
このファイルには、秘密鍵から作られた公開鍵と秘密鍵のハッシュ値（鍵の情報を要約した物）が入っています。さらに証明書に記載する署名情報（サーバのFQDNとか組織名とか）が加わります。

```sh 
$ openssl req -new -key server.key > server.csr
```
このときに、サーバのFQDNや組織名、所在地など聞かれます。適当で良いようですが、サーバのFQDNはきちんと入れておいた方が良いようです。証明書のFQDNとそれを設置したサーバのFQDNが違うのはダメなような気がしますね。

再び内容を確認します。

```sh 
$ openssl req -test < server.csr 
```

###3.認証局に成り代わって、証明書にサインします
できあがった証明書署名リクエストファイルに署名をして正式な証明書にします。  
本来これは認証局がやることですが、「おれおれ」なので「おれ」が証明書にサインします。

``` 
openssl x509 -req -days 7300 -signkey server.key < server.csr > server.crt
```
-days オプションでは有効期限を指定します。ここでは7300日、20年、だいたい私が死ぬまで有効。

できあがりを確認してみます。

```
$ openssl x509 -text < server.crt 
```
先ほど入力した組織名やFQDNが見えてくると思います。

必要なファイルは、.crtファイル（証明書）と.key（秘密鍵）です。
両方ともownをrootにして、パーミッションを600に設定しておきます。

## まずはwebサーバに設定してみる

先のwebcamの投稿でnginxをインストールしましたが、このサーバに作った鍵を設定して試してみます。

これも、設定方法を検索したところ、そのものずばり["nginxのTLS設定"](http://heartbeats.jp/hbblog/2012/06/nginx06.html)というページが見つかりました。この連載、とてもわかりやすくnginxの設定方法が書かれていますので、あとでよく勉強しておこうと思います。

やることとしては、configファイルを変更してhttpsの受け口を作り、そこに先ほど作った証明書をいれる、ということになります。

設定ファイルは、先のページを参考に以下の様にしました。

```sh /etc/nginx/sites-sites-available/default
server {
	listen 443 ssl;
	server_name my.www.server.jp;
	root /home/mynginx/www;
	index index.html index.htm;

	ssl_certificate /etc/nginx/server.crt;
	ssl_certificate_key /etc/nginx/server.key;

	ssl_session_timeout 5m;
	ssl_session_cache shared:SSL:10m;

	ssl_protocols SSLv3 TLSv1;
	ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv3:+EXP;
	ssl_prefer_server_ciphers on;

	location / {
		try_files $uri $uri/ =404;
		auth_basic "Restricted";
		auth_basic_user_file /etc/nginx/.htpasswd;
	}
}
```
* httpsのポートを443に設定してsslを有効にします。
* サーバのFQDNを設定
* webサーバのドキュメントルートを設定
* 証明書と秘密鍵を指定
* タイムアウトとキャッシュを設定（ここら辺は参考webページの写しです）
* SSLのプロトコル指定と暗号化スイートの指定です。ここら辺はnginxのデフォルト設定ファイルのコピー
* ドキュメントの特定の場所のビヘイビアの指定です。ここではルート以下の全てのアクセスでBasic認証を要求するような設定です

認証のために.htpasswdが必要になりますが、これはhttp-toolsのなかにあるhtpasswdコマンドでつくりました。

```sh how-to make a password file
$ sudo htpasswd -c .htpasswd UserName
New password:
Re-type new password:

```
のようにしてパスワードファイルを作成して、設定します。


###4.設定を有効にして、再起動

設定を書き終えたら設定を確認して、読み込ませます。

```sh restart nginx
$ sudo nginx -t
$ sudo nginx -s reload
```

###5.動作確認
これで、TLSが有効になっているはずです。アクセスしてみます。

`https://my.www.server.jp`

ブラウザからは「この証明書は無効です」などのワーニングが出てきました。証明書に有効なサインがない、サインした人が「ちゃんとした」人じゃ無いので、このようにワーニングがでます。
出てきたワーニングから「証明書を確認する」などのボタンをおして、自分が作った証明書だということを確認します。

確認できたら、ワーニングを無視して進みます。
ここでログイン（Basic認証）のポップアップが出てくるはずです。先ほど設定したログイン名とパスワードを入力します。無事ログインして、webページがみれました。



