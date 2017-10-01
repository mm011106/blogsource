---
title: Gitlabのinstall
date: 2017-09-16 10:50:06
categories: Gitlab docker
tags:
---
Gitlabをインストールします。

といってもコンテナをプルしてくるだけですけど。

<!-- more -->

```
docker pull gitlab/gitlab-ce

sudo docker run --detach --hostname localhost \
--publish 10443:443 \
--publish 10080:80 \
--publish 10022:22 \
--name gitlab_test --restart always \
--volume /srv/gitlab/config:/etc/gitlab \
--volume /srv/gitlab/logs:/var/log/gitlab \
--volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:latest


```
で、一応動く。ポートを10080に設定したので、
`http://localhost:10080/`でアクセスしてみる。

最初、「応答が遅くなっています」というエラーメッセージがでるが、しばらく待って再度アクセスするとgitlabが見えるようになる。


## おまけ: docker-composeを入れる

最新バージョンの番号を[ここ](https://github.com/docker/compose/releases)で確認して、必要なものを入れる。
今回、1.16.1が最新版だったので、下記のようにした。

```
 sudo curl -L \
    https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m`  \
    -o /usr/local/bin/docker-compose
```

