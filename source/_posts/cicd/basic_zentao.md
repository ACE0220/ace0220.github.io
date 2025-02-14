---
title: 测试环境搭建（四）-docker&zentao（禅道）
date: 2021-07-03 09:58:30
categories:
  - cicd
tags:
  - docker
  - linux
  - ubuntu
  - mysql
  - redis
---

禅道是一个国产开源项目，支持项目集管理、测试管理，devops等功能，支持Scrum、瀑布、看板、IPD、融合敏捷、融合瀑布模型，测试团队中常用的敏捷模型也能很好地支持。
<!-- more -->

# 一、拉取镜像

禅道有自己的docker镜像仓库，通过禅道的仓库拉取镜像

```
$ sudo docker pull hub.zentao.net/app/zentao:latest
```

# 二、运行镜像

官方文档提到持久化的数据都保存到容器的/data目录，那我们需要持久化，只需要将持久化的目录挂载到容器/data

宿主机创建持久化目录并赋予权限

```
$ cd ~
$ sudo mkdir -p zentao/data
$ sudo chmod -R a+rwx zentao
```

前面我们搭建了mysql和redis，禅道也要使用到这两个工具

```
$ sudo docker run -d --name zentao_test \
--link test_mysql \
--link redis_test \
-e ZT_MYSQL_HOST=test_mysql \
-e ZT_MYSQL_PORT=3306 \
-e ZT_MYSQL_USER=root \
-e ZT_MYSQL_PASSWORD=abc123456 \
-e ZT_MYSQL_DB=zentao_test \
-e PHP_SESSION_TYPE=redis \
-e PHP_SESSION_PATH=tcp://redis_test:6379?auth=abc123456 \
-v ~/zentao/data:/data \
-p 8088:80 \
hub.zentao.net/app/zentao:latest
```

```
# 参数解释
--link test_mysql 同一个docker内，可以通过容器名称获取到host,获取到test_mysql的host
# 环境变量
ZT_MYSQL_HOST MySQL 主机地址
ZT_MYSQL_PORT MySQL 主机端口
ZT_MYSQL_USER MySQL 用户名
ZT_MYSQL_PASSWORD MySQL 密码
ZT_MYSQL_DB MySQL 数据库名称
PHP_SESSION_TYPE php session 类型，files | redis
PHP_SESSION_PATH php session 存储路径
```

执行docker run 之后，我们查看一下禅道的logs，容器启动完成，禅道的logs指引我们要从浏览器打开ip:port开始安装向导。
我们禅道docker容器对外暴露了8088端口，虚拟机默认会对外暴露8088端口，那么访问虚拟机ip:8088直接就可以访问到docker的8088
获取虚拟机的ip地址，不记得如何查看[点击这里，mysql篇的第五点](/cicd/basic_mysql/#五、物理机（笔记本或台式电脑）测试连接虚拟机的3306端口)

```
$ sudo docker logs 禅道容器id
12:03:31.20
 12:03:31.21 Welcome to the Easysoft ZenTao 21.4 container
 12:03:31.21 Subscribe to project updates by watching https://www.zentao.net
 12:03:31.22 Submit issues and feature requests at https://www.zentao.net/ask.html
 12:03:31.23
 12:03:31.27 INFO  ==> Prepare persistence directories.
 12:03:32.33 INFO  ==> Render php.ini with environment variables.
 12:03:32.39 INFO  ==> Check whether the Redis is available.
 12:03:33.41 INFO  ==> Init: Redis is ready.
 12:03:33.43 INFO  ==> Check zentao data owner...
 12:03:33.45 INFO  ==> Render apache sites config with envionment variables.
 12:03:33.49 INFO  ==> Prepare custom extensions.
 12:03:33.53 INFO  ==> Check whether the MySQL is available.
 12:03:33.55 INFO  ==> Check whether the Apache is available.
 12:03:34.56 INFO  ==> Apache: MySQL is ready.
 12:03:34.58 WARN  ==> Sentry: Waiting Apache 1 seconds
 12:03:34.61 WARN  ==> Database zentao_test does not exist, creating...
 12:03:34.68 INFO  ==> Database zentao_test created successfully
[Fri Feb 14 12:03:34.913841 2025] [mpm_prefork:notice] [pid 183:tid 183] AH00163: Apache/2.4.62 (Unix) OpenSSL/1.0.2k-fips SVN/1.14.4 configured -- resuming normal operations
[Fri Feb 14 12:03:34.914392 2025] [core:notice] [pid 183:tid 183] AH00094: Command line: '/opt/zbox/run/apache/httpd -D FOREGROUND'
 12:03:36.60 INFO  ==> Sentry: Apache is ready.
 12:03:36.64 INFO  ==> The service has been started. Open your browser to access the specified domain or ip:port to complete the installation wizard
 12:03:36.65 INFO  ==> 服务已启动完成, 请使用浏览器访问设置的域名或ip:port, 继续完成后续安装向导
 12:03:38.68 INFO  ==> The service has been started. Open your browser to access the specified domain or ip:port to complete the installation wizard
 12:03:38.69 INFO  ==> 服务已启动完成, 请使用浏览器访问设置的域名或ip:port, 继续完成后续安装向导
 12:03:41.73 INFO  ==> The service has been started. Open your browser to access the specified domain or ip:port to complete the installation wizard
 12:03:41.74 INFO  ==> 服务已启动完成, 请使用浏览器访问设置的域名或ip:port, 继续完成后续安装向导
 12:03:42.77 INFO  ==> The service has been started. Open your browser to access the specified domain or ip:port to complete the installation wizard
 12:03:42.78 INFO  ==> 服务已启动完成, 请使用浏览器访问设置的域名或ip:port, 继续完成后续安装向导
```

# 三、安装向导

首先我们打开浏览器输入ip:port，作者虚拟机地址是192.168.53.131，端口指定了8088，打开之后就发现了禅道的安装向导

![](/pics/testing/zentao_1.png)

接来下就是一直下一步直到检查环境
session目录显示存储目录不存在，不可写，但是检查又通过，因为我们在docker run zentao的时候设置了PHP_SESSION_TYPE=redis而不是files，所以检查通过，点击下一步

![](/pics/testing/zentao_2.png)

接着禅道开始自动往我们自己配置的mysql和redis中写入表和需要的数据

![](/pics/testing/zentao_3.png)
![](/pics/testing/zentao_4.png)

数据库表安装完毕后，禅道会提示我们配置文件所在路径，这个路径是位于docker容器内的，不要搞错了

![](/pics/testing/zentao_5.png)

剩余部分只要跟着提示一步步操作即可，并没有什么难处，这里不再细说。

# 四、检查禅道是否使用上了我们部署的外置mysql和redis

还是老习惯使用navicat进行检查，可以发现，mysql数据库创建和数据表安装，redis中也存储了我们按照提示填的相关信息。
说明外置mysql和redis在禅道中已经生效了。

![](/pics/testing/zentao_6.png)
![](/pics/testing/zentao_7.png)


# 五、当前阶段小总结

整个测试环境简单搭建基于虚拟机和docker，再到数据库和项目管理系统，在真实测试环境中这还显得非常不足。万事开头难，无论什么都是一步一步的从基建开始。不懂的地方也还有很多，继续加油进步吧~

参考资料：
https://www.zentao.net/price.html 功能对比
https://www.zentao.net/book/zentaopms/docker-1111.html 禅道docker安装
https://www.zentao.net/book/zentaopms/docker-1111.html#9 环境变量