---
title: gost 配置方案
date: 2019-11-22 13:03:46
tags: 
	- gost
	- ubuntu
	- systemd
	- proxy
---

# 配置 gost 开机自启

[gost](https://github.com/ginuerzh/gost)是GO语言实现的安全隧道，支持多种网络协议，对多平台有着很好的支持。本文将gost配置为系统服务，从而实现开机自启。

ubuntu系统中，开机自启有很多种实现方案，比如Startup Application, 或者init.d中配置开机脚本，不过ubuntu 16.04之后的版本，比较推荐使用systemd进行管理。systemd 在[阮一峰的文章](## 参考链接)中已经有详细的介绍，此处不做赘述，直接介绍配置的具体细节。

## 配置方法

第一步，首先下载最新的 gost，并拷贝到/usr/bin/目录下

第二步，在/etc/systemd/system/目录下新建名为gost.service的文件，文件具体内容如下

```shell
[Unit]
Description=GOST SERVER
After=network.target

[Service]
Type=simple
EnvironmentFile=/path/to/enviroment/file
ExecStart=/usr/bin/gost -L "http2://${USER}:${PASS}@${DOMAIN}:${PORT}?cert=${CERT}&key=${KEY}&probe_resist=code:404"
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

其中USER PASS等变量定义在 EnvironmentFile 中，EnvironmentFile 的格式与shell 脚本类似

最后，依次运行以下命令启动服务

```shell
# 重新加载配置文件
systemctl daemon-reload
# 启动服务
systemctl start gost
```

## 进一步改进

首先安装Json解析器 jq
```shell
sudo apt-get install jq -y
```
然后使用以下脚本自动检测gost更新，下载最新release的gost，使用该脚本前，
```shell
#!/bin/bash
cd $(dirname $0)
platform='linux-amd64'
current_version=$(cat current_version)
latest_version=`curl -s https://api.github.com/repos/ginuerzh/gost/releases/latest | jq -r ".name"`
name=`curl -s https://api.github.com/repos/ginuerzh/gost/releases/latest | jq -r ".assets[] | select(.name | test(\"${platform}\")) | .name"`
url=`curl -s https://api.github.com/repos/ginuerzh/gost/releases/latest | jq -r ".assets[] | select(.name | test(\"${platform}\")) | .browser_download_url"`
echo $url
echo $platform
echo $latest_version
echo $current_version
echo $name
if [ ! -d ${HOME}/Downloads" ]; then
  mkdir ~/Downloads
fi
if [ "$current_version" != "$latest_version" ]; then
  wget $url -O ~/Downloads/$name
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

