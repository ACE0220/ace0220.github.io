---
title: github-page部署https
categories:
  - github
tags:
  - github-page
  - https
date: 2021-05-20 16:53:00
---

个人域名接入github，强制使用https。

<!-- more -->

# 前置

在做以下事情之前，先要保证username.github.io是可以正常访问的，也就是用户的github-page要可以正常访问

在github page部署个人blog也很多教程了，这里不再细说。

## 个人域名

个人购买域名可以通过各个服务商进行购买，阿里云、腾讯云、华为云等等，并且网上也很多教程，这里不再细说了。

## CNAME

笔者的blog是基于[hexo-theme-melody](https://molunerfinn.com/hexo-theme-melody-doc/zh-Hans/)

CNAME文件需要放在source下，在github action部署的时候会自动将CNAME部署到gh-page分支

### CNAME文件内容

在source下创建CNAME文件，写入域名，yourdomain.cn

**CNAME文件名要大写**

**CNAME文件名要大写**

**CNAME文件名要大写**

source/CNAME

```text
yourdomain.cn
```

# 域名商管理后台修改域名解析

默认情况下域名是解析到购买的服务器，现在我们要修改到指向username.github.io

- 添加一个CNAME，主机记录@，记录值username.github.io
- 添加一个CNAME，主机记录www，记录值username.github.io

这样无论是否使用www，都能正常访问，www.yourdomain.cn, yourdomain.cn，都会先解析成username.github.io，再根据CNAME变回个人域名

![](/pics/github/deploy-https-1.png)

# github setting

去github对应的blog仓库内设置个人域名和https事宜

打开仓库，点击Settings，点击左侧的Pages，找到custom domain，填入个人域名

如果域名管域名解析那里设置正确，填入个人域名后，稍等一会会显示dns解析正确，就可以勾选Enforce HTTPS了

![](/pics/github/deploy-https-2.png)
![](/pics/github/deploy-https-3.png)

# ping yourdomain.cn

```sh
64 bytes from xxx.xxx.xxx.xxx: icmp_seq=0 ttl=53 time=99.820 ms
64 bytes from xxx.xxx.xxx.xxx: icmp_seq=1 ttl=53 time=100.451 ms
64 bytes from xxx.xxx.xxx.xxx: icmp_seq=2 ttl=53 time=101.277 ms
64 bytes from xxx.xxx.xxx.xxx: icmp_seq=3 ttl=53 time=109.720 ms
```

上方打印的 from xxx.xxx.xxx.xxx，这些xxx代表的是ip地址。

如果这个ip地址并不是你的服务器地址，并且个人域名可以正常访问，说明配置没有问题了。

