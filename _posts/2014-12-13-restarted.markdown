---
layout: post
title: "やっと復活。。。"
date: 2014-12-13 16:23:35 +0900
comments: true
categories: [ubuntu, octopress] 
---

前のブログで「ランチャが消えちゃう」なんていう話をしていたらもっと深刻な状況に。  
今度は、ランチャやメニューは表示されるのですが、デスクトップ自体が真っ黒のままで何も表示されず。ターミナルのアイコンをランチャ上でクリックすると、ターミナルは立ち上がっているようなのですが、見えません。メニューも一応見えていますが、クリックすると茶色いプルダウンらしきものが広がるだけで何も見えません。
<!-- more -->

ブログに書いた手順を試してみるも、ターミナルが見えないのでいかんともし難く。。。。  
ブラインド状態でコマンド入力してみても、うまく入力できているようですが、効果なし。

---

##新しいubuntuを立ち上げ
webでいろいろ調べましたが、結局解決には至らず。新しいubuntuをインストールするのが早いのじゃないか、ということで再インストールに。

幸い、MacOSのParallelsの上の仮想PCだったので、トラブった環境はそのままにしてすすめました。こういうとき仮想環境は楽ですね。

まずは、大事なブログのソースファイルをまるごと救出。  
Ctrl-Alt-F1でCUIベースのターミナルを開いて、

	tar czf octopress.tar.gz octopress

のような感じで。

さて、これをscpでコピーしようとしたら、どうもうまくいかず。。。

きちんとsshの設定をしていなかったからか。。。反省。

ですが、これまたラッキーなことにParallelsにはホストOSとのファイル共有が標準で用意されていて、これがubuntuでも有効です。

	cp ./octopress.tar.gz ~/Desktop/Parallels\ Shared\ Folders/

とするだけです。
これで大事な原稿と環境はホストOSにコピーされました。

肝心のインストールは、ubuntu のインストール用ディスクイメージを持ってきて、Parallelsに教えてあげるだけであとは勝手にインストールして完了。

---
##git インストール

さてここから環境の構築です。

まず、SSH関係をしっかりと構築したあと、

[octopressのセットアップ手順](http://octopress.org/docs/setup/)にしたがって、

	sudo apt-get install git

としてインストール。簡単。問題無し。

一応、動作を確認。gitで使うディレクトリに移動して

	git init

エラーなし。okです。


##ruby インストール

これまた、手順にしたがって、rbenvでRubyを[インストール](http://octopress.org/docs/setup/rbenv/)することにします。

```sh
cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
source ~/.bash_profile 
```

ubuntuでは、`.bash_profile`をログイン時に実行するために

	echo 'source ~/.bash_profile' >> .bashrc

としました。

インストールの結果を確認します。
新しいターミナルを開いて、

	echo $PATH

の結果に`~/.rbenv/bin`などが入って入ればいいかと思います。

これでrbenvツールが設定できたので、ここからRubyのインストールです。

私の環境では`zlib1g-dev`と`libssl-dev`が入っていなかったので、あとあとエラーしました。

念のため前もって

	sudo apt-get install zlib1g-dev libssl-dev

などとして、インストールするか確認しておいたほうがいいかもしれません。
そうでないと何回もRubyをインストールすることになります。

- octopressのセットアップではRubyのモジュールをダウンロードしますが、このときサーバにSSLで接続します。そのためにSSLのライブラリが必要なようです。

では、インストール。解説にしたがい

```sh
$ rbenv install 1.9.3-p0
$ rbenv local 1.9.3-p0
$ rbenv rehash 
```

と、しました。

実は、`rbenv install 1.9.3-p0`で`installなんてコマンドないよ！`と起こられましたが、`ruby-build`が正しい位置に持ってこれていなかったのが原因でした。

	git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

を正しく実行し直して事なきを得ます。

- webの解説によれば、`rbenv local ruby 1.9.3-p0`は、その他のディレクトリにあるRuby環境を変更することなく、octopressをインストールするために使われるそうです。もし、どこでもこのバージョンを使いたいなら、local を grobal に変更するといい、と書かれています。

インストールができたら、`Ruby --version`でバージョンを確認。OK。

---
## Octopress
[こちらの手順]()にしたがってインストール。

まず

```sh
git clone git://github.com/imathis/octopress.git octopress
cd octopress 
```
としてoctopress自身を持ってきます。

さらに、gemで必要なRubyモジュールをインストール
```sh
gem install bundler
rbenv rehash # If you use rbenv, rehash to be able to run the bundle command
bundle install 
```

ここで、ちゃんと動くかどうか

	rake generate

でためしてみたところ。

```sh
$ rake generate
## Generating Site with Jekyll
identical source/stylesheets/screen.css  
/home/masa/.rbenv/versions/1.9.3-p0/lib/ruby/gems/1.9.1/gems/  
execjs-2.2.2/lib/execjs/runtimes.rb:51:in `autodetect': Could not find   
a JavaScript runtime. See https://github.com/sstephenson/execjs for a   
list of available runtimes. (ExecJS::RuntimeUnavailable)

```
と言われてしまいました。
"octopress Could not find a JavaScript runtime"をキーワードに
googleさんに尋ねてみると、解決策が

	sudo apt-get install nodejs

を実行しろ、ということだったので、さっそくインストールし、無事`rake generate`できることを確認。

さらに`rake preview`で出来栄えも確認し、インストール完了。

やっと環境が整いました。

---

## 昔の機械から原稿を持ってくる

さて、昔の機械からデータを持ってこなきゃいけませんが、オリジナルのoctopress からテーマを更新(oct2を使用)しているので、もしかしたらコピーだけではだめかもしれないという危惧もあり、厄介かなあとおもいつつも、ダメモトということで、octopressディレクトリをまるごとコピー。

オリジナルをキープしておいて、持ってきたものをコピーしましたが、`rake generate`では`「バージョンが違いますけど！」`と注意されてしまいます。しかたがないので、オリジナルに戻して、今度はsourceフォルダだけを昔のものに入れ替えて試してみました。

すると、何事もなくgenerateが可能に。さらにpreviewして元のBlogページが戻ることを確認！！ああ、素晴らしい。

ということで、記事を書き始めるとまたまたトラブル。。。。

どうも文字が大きい。さらに、バックグラウンドの絵がページの頭の部分で黒くなる。。。。

「うむ、やはりコピーだけではダメかああああ。」
しかたなく、もう一度octopressの環境から作り直し。さらにoct2も素直にインストールし直し。

まず、octopressのディレクトリをまるごと削除。もう一度インストールし直します。

さらにoct2を以下のようにしてインストール
```sh
git clone https://github.com/bijumon/oct2.git .themes/oct2
rake install["oct2"]
```

[oct2のページ](https://github.com/bijumon/oct2)によるインストール方法ではcloneの保存先が`.themes/`ですが、これではエラーしてしまうので、`.themes/oct2`としてみました。

これで一応環境は整っているはずなので、このまま何もせず

	rake preview

としてfirefoxなどで[localhost:4000](http://localhost:4000)をアクセス。oct2が有効になっていることを確認します。

いよいよblogのソースをコピーします。といっても、.mdの原稿を`_post`ディレクトリにコピーするだけです。

さらに、以下のファイルをコピーしました。

	./_config.yml
	./public/image/background.png  (自分で作ったバックグラウンドカラー）
	./public/image/logo.png
	./public/image/(その他必要な画像）
	./source/includes/post/date.html（日付表示がうまくいくように変更したもの）
	./source/(その他、自分で作ったディレクトリ）

これで、再び`rake preview`を実行。

！！！やっとこれで元通り！！！とおもったところが、まだ文字が異常に大きい。。。

これまた、googleにたずねると、「[ここをご覧なさい](http://blog.nambo.jp/2013/07/26/change-octopress-font-size/)」といわれたので、そのまま追加して、納得できる出来栄えに。。。。
```css ./sass/custom/_stypes.scss　に追加
h1 { font-size: 1.2em;}

h2 { font-size: 1.1em;}

h3 { font-size: 1.0em;}

#content .blog-index {
  article h1 {
    font-size: 1.2em;
  }
}
```
これもちょっと腑に落ちない妙な動作をするのですが、まあ、とりあえず吉としておきます。とおもいましたが、やはり気になるので[他の方法](http://yuyunko.github.io/blog/2013/03/04/octopress-%E3%83%86%E3%83%BC%E3%83%9E%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%B9%E3%82%99/)も試してみるとこちらの方がいい感じなので、こうしました。
```css ./sass/custom/_styles.scss　を変更
body { font-size: 0.8em; }
```

お疲れさまでした。
ほぼ1日消費です。

そうそう、Parallelsでスナップショットをとっておこう。

### 2014/12/14 update

_styles.scssをエディットしましたが、どうもこれが有効になったり無効になったり、おかしな感じになります。ちょっとしらべたところ、geditでは編集しているファイルのバックアップファイルを編集しているファイルと同じホルダに作ります。この時のファイル名が　"オリジナルのファイル名～"　となるのですが、どうもそれをoctopressが区別できていない、あるいは編集中のものにはアクセスロックがかかる？ようです。

geditを使うときは、保存したファイルを一旦閉じるか、バックアップファイルを作るのをやめるように設定するか、編集後バックアップファイルを消して　generate　なり　preview　なりしたほうがいいようです。




