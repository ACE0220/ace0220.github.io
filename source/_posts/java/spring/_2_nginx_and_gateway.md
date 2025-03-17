---
title: spring体系开发-接入层-nginx+spring cloud gateway
date: 2025-3-15 09:58:30
categories:
  - java
  - spring
tags:
  - java
  - mall
  - spring
  - SSL
  - nginx
---
mall
<!-- more -->
# docker网络

```bash
sudo docker network create spring_net
```

# nginx部署

## 将配置文件复制到宿主机

先将默认的配置文件和文件夹复制出来

```bash
# 创建文件夹，赋予读写权限
sudo mkdir -p ~/nginx/{conf,conf/conf.d,log,html}
sudo chmod -R a+rw nginx
# 运行一个基础nginx基础镜像，为了复制配置文件出来到宿主机
# docker不支持直接挂载文件，宿主机必须先存在文件，才能与docker进行挂载
sudo docker run -d --name nginx nginx:stable-perl
sudo docker cp nginx:/etc/nginx/nginx.conf ~/nginx/conf/nginx.conf
sudo docker cp nginx:/etc/nginx/conf.d ~/nginx/conf/
sudo docker cp nginx:/usr/share/nginx/html ~/nginx/
# 获取container id，停止容器并删除
sudo docker ps -a
sudo docker stop <container id>
sudo docker rm <container id>
```

## 运行真正的容器

运行容器并检查，检查可以使用浏览器，作者懒得截图了，就用curl，返回了html文档

```bash
# 运行容器
sudo docker run \
--name nginx \
--restart=always \
--network=spring_net \
-p 80:80 \
-p 443:443 \
-v ~/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v ~/nginx/conf/conf.d:/etc/nginx/conf.d \
-v ~/nginx/log:/var/log/nginx \
-v ~/nginx/html:/usr/share/nginx/html \
-d nginx:stable-perl
# 检查
sudo docker ps -a
curl http://localhost
```

curl返回值

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

# spring cloud gateway




参考资料：
https://qizhanming.com/blog/2015/12/23/openssl-ecdh-rsa-certificate 自签证书

https://www.cnblogs.com/wshenjin/p/12519455.html 自签证书配置

https://cloud.tencent.com/developer/article/2350984 nginx和gateway 概念

https://blog.csdn.net/qq_33192671/article/details/144433363 基础配置