---
title: gost 自动更新
date: 2020-06-08 13:03:46
tags: 
	- gost
	- ubuntu
	- systemd
	- proxy
---

# gost 自动更新

在上一篇文章中，我们介绍了怎样实现gost的自动启动：[配置gost开机自启](https://blog.yu-xiaoxian.me/2019/11/22/Mannual/install-gost-as-system-service/)。但是 gost 无法通过`apt-get upgrade`自动更新，因此我开发一个脚本实现 gost 自动更新，保持安装的是最新的gost，从而避免版本老旧引起的安全问题。

首先安装Json解析器 jq
```shell
sudo apt-get update
sudo apt-get install jq -y
```
然后使用以下脚本自动检测gost更新，下载最新release的gost
```shell
#!/bin/bash
cd $(dirname $0)
if [ ! -f current_version ]
  touch current_version
fi
url="https://api.github.com/repos/ginuerzh/gost/releases/latest"
platform='linux-amd64'
current_version=$(cat current_version)
latest_version=`curl -s $url | jq -r ".name"`
name=`curl -s $url | jq -r ".assets[] | select(.name | test(\"${platform}\")) | .name"`
download_url=`curl -s $url | jq -r ".assets[] | select(.name | test(\"${platform}\")) | .browser_download_url"`
echo $download_url
echo $platform
echo $latest_version
echo $current_version
echo $name
if [ ! -d ${HOME}/Downloads" ]; then
  mkdir ~/Downloads
fi
if [ "$current_version" != "$latest_version" ]; then
  wget $download_url -O ~/Downloads/$name
  gunzip -fk ~/Downloads/$name
  cp ~/Downloads/${name%.*} /usr/local/bin/gost
  echo $latest_version > current_version
fi
```
然后使用`crontab -e`，在其中写入，就可以定期执行gost啦
```shell
0 4 * * * service gost stop
1 4 * * * sh <path-to-shell>
2 4 * * * service gost start
```

# 参考链接

1. [HOW TO DOWNLOAD THE LATEST RELEASE FROM GITHUB](https://starkandwayne.com/blog/how-to-download-the-latest-release-from-github/)
