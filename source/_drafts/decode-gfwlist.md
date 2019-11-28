---
title: 解码gfwlist.txt
tags:
	- shell 
	- decode 
---

想要科学上网的话，一个很便捷的配置就是gfwlist，目前该列表已经在Github开源并持续维护[gfwlist]( https://github.com/gfwlist/gfwlist )，但是这个文件直接打开的话是一堆ASCII码，这里提供一种shell命令行的解码方式

```shell
curl https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt | base64 -d > gfwlist.txt
```

这时打开文件，会发现已经全部转成明文了。因为这个文件是base64编码的，所以我们使用base64进行解码，就可以了



## 参考链接

1. [Base64笔记]( http://www.ruanyifeng.com/blog/2008/06/base64.html )
2. gist