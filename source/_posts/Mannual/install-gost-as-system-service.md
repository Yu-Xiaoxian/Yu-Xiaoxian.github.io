---
title: 配置gost开机自启
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
EnviromentFile=/path/to/enviroment/file
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

研究怎样自动更新gost到最新版本

# 参考链接

1. [阮一峰：Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)
2. [阮一峰：Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)