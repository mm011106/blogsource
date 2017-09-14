---
layout: post
title: "TrelloにPythonでアクセスしてみる"
date: 2015-12-06 10:08:06 +0900
comments: true
categories: Trello, python
---

IoTとは全然関係ないですけど、TrelloというSaaSを業務で使い始めて、他のツールとのインターフェイスが必要になり始めました。Trelloはカード型のToDo管理のシステムですが、うちではワークフローのステート管理につかっています。ですので、Trelloで管理しているワークフロー終了後に次の別のワークフローに移すことやチェックのためにデータをダウンロードする必要が出てきました。

TrelloはJSON形式でデータを一括ダウンロードできるのと、直接APIを通じてデータにアクセスすることもできます。
今回はPythonでAPIを叩くことに挑戦してみました。

<!-- more -->

## 準備
先人のおかげさまで、TrelloへのアクセスにもPythonのパッケージが用意されています。結果として、ほとんどコードを書かずにアクセするこことができました。  
MacOS10.9にCanopyをインストールした環境での作業記録になります。

パッケージ`trello 0.9.1`のダウンロードはPythonSoftwareFoudationの[ページ](https://pypi.python.org/pypi/trello)から。

さらに必要であればURLリクエスト処理をしてくれる`requests`をインストールしてください。（私の場合は、新品に近い環境でしたので、インストールが必要でした）

インストールには、ダウンロードしたフォルダに移動して`$ python setup.py install`です。

さて、実際のコードはどうなるのかと思い、参考のために
[Trello Python API](https://pythonhosted.org/trello/index.html)
に行って例をみてみると、たった数行でアクセスできることが解りました。

この例ではパブリックにオープンなボードでの例ですが、今回は公開されていない（パーソナルな）Trelloボードにアクセスすることを試します。  
Personalなボードにアクセスするため、下記の手順でユーザに対応したAPI-keyとログインを省略するためのアクセストークン(以後token)を入手する必要があります。  
私の場合は両者とも手作業で行いましたが、tokenについてはスクリプト(pythonから）で入手できるかもしれません。

### API keyを入手する
- Trelloにログインします。
- ログインした状態で　[https://trello.com/app-key](https://trello.com/app-key)にアクセスすると、’key'と'Secret'という２つの番号をもらえます。
- 'key' はスクリプトの中で使用します。'Secret'は大事にしまっておきなさい、ということなので、しまっておきます。

### tokenを入手する
pythonを起動して
```
from trello import TrelloApi	
trello = TrelloApi('key')  # 先ほどゲットしたAPI-keyを入れます
trello.get_token_url('My App', expires='30days', write_access=True)
```
とすると、urlが表示されますのでそれをブラウザで表示させます。

「このMy Appというアプリケーションにアクセスを許可しますがいいですか？」的な確認が出てきて、okするとtokenが表示されると思います。それを記録しておきます。


## 実行
ここまでくればあとはコードを書くだけです。
今までのところで、アクセスに必要な情報とパッケージはok準備万端なので、例などを参考に下記のコードをステップバイステップで実行してみました。

```
from trello import TrelloApi
trello = TrelloApi('key')
trello.set_token('Token')

token='****'  #記録したtokenとkeyを指定します。
key='***'

#これでアクセス可能な状態になります。
# 例として、ボード上のリスト名とそのidを表示させます。
# （APIでは常にidでアクセスする先を指定するので重要）

idBoard='****'  #ボードidを入れます。

lists=trello.boards.get_list(idBoard)
for list in lists:
    print list['name'] + "    " + list['id']

```
たった数行でアクセス可能になり、実際のアクセスも１行でできるというコードに。**すばらしい！**

実は、実際のアクセスにはボードやリスト、カードなどのidが必要です。上記の例ではボードidは既知としてコードを書いています。

実際はAPIから引っ張れるかもしれませんが、そこまでたどり着けていません。idは直接Trelloのページからボードに移動し、[menu]-[more]-[print & export]-[export to JSON]でデータを表示させ、そこから読み取っています。

JSONをブラウザでみるためにChromeに[JSONView](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc)を入れています。

すばらしく便利です。



