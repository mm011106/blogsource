---
title: dockerCEのインストール
date: 2017-09-16 10:11:21
categories: docker gitlab
tags: docker
---

２年前に立ち上げたローカルのgitlabが古くなっているようなので、新しくしたいと思っているのですがまずはどんな感じなのか確認するために、Docker環境を作りなおすところからはじめました。
ここではDockerCEの環境構築まで。

<!-- more -->

# まずはDocker環境
今回つかうのは　Docker CEという２０１７年のバージョン。以前のDocker EngineがアップデートされてCEになった。

[ここを参考に](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)


## 前バージョンを削除


```
sudo apt-get remove docker docker-engine docker.io
```

## インストール準備

リポジトリから、手動で、テスト用に、という３つの方法が提案されていますが、ここではパッケージリポジトリから導入することにします。
まずは、現状インストールされているパッケージを最新に。

```
sudo apt-get update
```

DockerCEをインストールするためにいくつかの関連したパッケージをインストールする必要があるので、まずはそれをインストール。私の場合、最初からインストールされていた。


```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

Dockerから鍵をコピーしてくる。

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

ａｐｔの鍵が追加されるので、フィンガープリントを確認。

```
$ sudo apt-key fingerprint
/etc/apt/trusted.gpg
--------------------

 :
 :

pub   4096R/0EBFCD88 2017-02-22
      フィンガー・プリント = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22


```

ａｐｔに参照するべきリポジトリを追加します。
リリースにはバージョンと別にStable, Edge, testingというカテゴリがあるらしいが、ここでは普通にstableを入れる指定。

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

## インストール

いよいよインストール。

```
sudo apt-get update
sudo apt-get install docker-ce
```

動作確認
```
sudo docker run hello-world
```

## 片付け方

Uninstall the Docker CE package:

```
$ sudo apt-get purge docker-ce
```

イメージとか、コンテナとか、ボリウムとかカスタマイズされた設定ファイルは削除されない。
もし、これらをいっぺんに削除するなら、

```
$ sudo rm -rf /var/lib/docker
```

その他設定ファイルは手動で削除しないといけません。




