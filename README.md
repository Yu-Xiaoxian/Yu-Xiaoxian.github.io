# [Yu-Xiaoxian's_Blog](http://blog.yu-xiaoxian.me)

[![Build and Deploy](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io/workflows/Build%20and%20Deploy/badge.svg)](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io/actions?query=workflow%3A%22Build+and+Deploy%22)
[![hexo](https://img.shields.io/badge/hexo-%3E%3D%204.2.0-blue.svg)](https://hexo.io)

[于小咸的日志](http://blog.yu-xiaoxian.me)

---

本博客使用Hexo生成静态页面，由Travis CI部署在Github Page上，并配置了自定义域名blog.yu-xiaoxiain.me。

1. [Hexo](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#1-hexo)
2. [Travis_CI](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#2-travis_ci)
3. [Github_Page](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#3-github_page)
4. [域名解析](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#4-域名解析)
5. [双线路解析](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#5-双线路解析)
6. [添加徽章](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#6-添加徽章)
7. [参考链接](https://github.com/Yu-Xiaoxian/Yu-Xiaoxian.github.io#7-参考链接)

## 1. Hexo

Hexo 是一个静态网页生成和部署的框架，可以简单理解为md文件到html的转换器，可在_config.yml文件中设置作者、主题等信息。

## 2. Travis_CI

Travis 是一款自动部署工具，而 Travis CI 是集成在Github上的在线解析工具。在项目分支的根目录下添加名为".travis.yml"的文件，该文件中包含了该项目部署的具体命令。

## 3. Github_Page

Github Page 是Github提供的静态网页部署平台，有“个人/组织页面”和“项目页面”两种。每个用户或组织只能有一个“个人/组织页面”，每个项目可以有一个“项目页面”。

## 4. 域名解析

域名解析分为两步，第一步是域名解析，第二步是301事件。

### 4.1 域名解析

域名解析是从个人域名<user-domain>到Github Page的解析，使得访问<user-domain>的时候，能够正确解析到Github Page，这需要在域名服务商和DNS服务商处设置。

### 4.2 301事件

这一步在Github Pages进行设置，是为了保证浏览器在访问Github Page<$user.name$.github.io>时，域名能够正确切换到个人域名。

具体做法是在master分支下添加名为CNAME的文件，文件中填写个人域名的url，以本博客为例，url为blog.yu-xiaoxian.me.

## 5. 双线路解析

## 6. 添加徽章

可以在项目的介绍文档中添加徽章，来说明项目的版本以及部署情况，本博客使用了Travis CI的生成徽章以及Hexo的版本徽章，更多徽章可以在网友[Bönnemann](https://github.com/boennemann)的项目[bades](https://github.com/boennemann/badges)中去查看。

## 7. 参考链接

1. [hexo博客搭建时遇到的一些问题](https://segmentfault.com/a/1190000003710962?_ea=336354#articleHeader3)
