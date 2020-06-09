---
title: 配置gost开机自启
date: 2019-11-22 13:03:46
tags: 
	- gost
	- ubuntu
	- systemd
	- proxy
---

## 进一步改进

首先安装Json解析器 jq
```shell
sudo apt-get update
sudo apt-get install jq -y
```
然后使用以下脚本自动检测gost更新，下载最新release的gost，使用该脚本前，
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
0 4 * * * sh <path-to-shell>
```

# 参考链接

1. [HOW TO DOWNLOAD THE LATEST RELEASE FROM GITHUB](https://starkandwayne.com/blog/how-to-download-the-latest-release-from-github/)
