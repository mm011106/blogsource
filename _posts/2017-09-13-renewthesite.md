---
title: "HTMLジェネレータを新しくしました"
date: 2017-09-13 20:35:25
tags: hexo
categories: hexo, git

---

今までoctpressを使っていましたが、HTMLジェネレータを新しくしました。
いろいろ調べて、HEXOというジェネレータが、テーマがいろいろ選べて良さそうで、生成も早いということでしたので選択。
こんな見た目になりました。

<!-- more -->

## 確かに早い
octpressでは新しいブログ原稿だけを選択的に変換するというのができなかったため、書き進めれば書き進めるほど生成に時間がかかるようになりました。エラーがあると修正サイクルが長くなって嫌になって結局書かなくなる、という感じでしたので、良かったです。気軽にかけるようになりました。
設定については後でまとめておきたいと思います。

みなさんのwebでは、「導入も超高速」という触れ込みのところが多かったですが、私はgithubへのデプロイがなかなかできずに苦労しました。色んな所で'git init'して、設定して、というのをやっているうちになんだか動いてしまいました。
エラーとしてはうまくローカル側でのユーザ設定ができていなかったということのようですが、一体どこのディレクトリの.gitを参照しているのかわからず盲滅法という感じでした。

まあ、完成したのでいいとしておきます。

## 細かいところがわからない
使い始めたばかりなので、写真を入れたりテーマのバックグラウンドの写真を変更したりとか、細かいカスタマイズポイントがわかっていません。おいおい調べて行きたいと思います。

## 写真の挿入

![iPhone X](/blog/image/01_s.jpg "cell phone")
hexo のブログソースのフォルダ(/source)の中に./image/というディレクトリを作ってその中に絵をおく。
それを
`
	![iPhone X](/blog/image/01_s.jpg "cell phone")
`
のように指定すればｏｋ．

ただ、サイズ指定はできない。
サイズ指定する必要があれば、
<img src="/blog/image/01_s.jpg" alt="iPhone X" title="cell phone" width="100">

`
<img src="/blog/image/01_s.jpg" alt="iPhone X" title="cell phone" width="100">
`

このブログはweb server ドキュメントルート直下ではなく/blogの下にあるのでイメージファイルのディレクトリは'/blog/image/'となる。

## ブログソースの保管
gitにやってもらう。

sourceディレクトリに移動して

```shell-session:git

git config --global user.email メールアドレス
git config --global user.name ユーザ名
git init
git remote add origin リモートリポジトリURLを指定

git add .
git commit -m "first update"
git push -u origin master
```

使うとき（読み込み）は
` git pull `

リモートリポジトリを更新したい時（保存）は
```
git add .
git commit -m "message"
git push -u origin master
```
  


