---
layout: post
title: "手始めに、色々と試してみます"
date: 2014-11-29 14:52:11 +0900
comments: true
categories: [日本語, Fromat Examples, test]
---

シェルコマンドの表示例です。

```sh
echo ${test}
cat /var/log/syslog | grep 'test_log' > temp.txt
```
<!-- more -->

```sh
rake watch
```

を別のコンソールから起動しておけば、ポスト用のテキストファイルを保存するたびにrake generateしてくれます。エラーも出るので構文がエラーしているとすぐフィードバックしてくれます。これは結構便利。

カテゴリは、カテゴリ名にスペースが入るとダメ（スペースで分けられて個別のカテゴリになってしまう）。スペースが入るときは[]で囲って、コンマで区切る。  
日本語も大丈夫。

改行するときは行末に２文字の半角空白を置くと、  
こんなふうに改行できます。

現在、octopressのディレクトリ内からソースファイルをエディットして、自動でgenerate、適当なところで手動でweb用のリポジトリ内にコピー、そこからcommit & pushしています。  
ちょっと効率が悪い、ソースがローカルに保存されてしまう、という欠点があるので何とかしたいと思います。  
世の中には rake setup_github_pages というものがあるらしいが、詳しく調べてみる。  
やはり大元の[ここ](http://octopress.org/docs/deploying/github/)が詳しいか？

写真
---

本題に入る前に、区切り線の挿入。
区切り線の前の行は文字が大きくなり、タイトルっぽくなります。  

大切な写真の埋め込み例をやります。  まずは、markdownの標準のやりかた  
```
![これは写真です](/images/hana.jpg "HANA!")
```

![これは写真です](/images/hana.jpg "HANA!")
写真の次のテキストはこんな位置に来ます。  


```
![これは小さい写真です](/images/hana_small.jpg "Shrinked HANA!")
```

![これは小さい写真です](/images/hana_small.jpg "Shrinked HANA!")

改行を１行入れたあとの、写真の次のテキストはこんな位置に来ます。  
次に、octopressでのタグフォーマットです。
フォーマットはこんな形になります。
```
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
```
class name の選択肢がわかりませんが、とりあえず例を参考にすると。。。。


```
{% img /images/hana.jpg 'HANA!' %}
```

{% img /images/hana.jpg 'HANA!' %}

このフォーマットだと左右に寄せることができ、サイズも指定できます。
```
{% img center /images/hana.jpg 400 'center 400 HANA!' %}
```

{% img center /images/hana.jpg 400 'center 400 HANA!' %}

すぐ次の行に文字を入れるとこんなかんじです。  
多分写真の下に文字が来ると思いますが。。。


```
{% img left /images/hana.jpg 300 '300 HANA!' %}
```

{% img left /images/hana.jpg 300 '300 HANA!' %}

写真の埋め込みコマンドの次の行です。ここに文章を入れないとすこしフォーマットがおかしくなる感じですね  
写真の縦の高さ分、文字がないと、次の写真を今の写真の右側に流し込もうとして変な感じになります。  

それは、それで正しいフォーマッティングかと思います。  
この行数では、safari上で行数が足りません。firefoxよりも文字が小さく表示されるようです。  
この状態で「区切り」を入れてみます。
---
やっぱりうまく行きませんね。

```
{% img right /images/hana_small.jpg 'Shrinked HANA!'　'ハナさん' %}
```

{% img right /images/hana_small.jpg 'Shrinked HANA!' 'ハナさん' %}

ここにも少し文章が必要です。  
ブログページの最後が写真で左右寄せに指定するときには、最後のところに少し文章が必要ですね。次のページのスタートが崩れます。  
  　　  
　　  
　　  
　　　　  

右寄せの写真、左寄せの写真を続けて挿入すると。。。。

{% img left /images/hana.jpg 200 'left HANA!' %}
{% img right /images/hana.jpg 200 'right HANA!' %}

そして、この文章は写真の次の行です。  
写真と写真の間に流し込まれているかしら？  




　　

　　

　　
　　








