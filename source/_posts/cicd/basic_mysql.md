---
title: 测试环境搭建（二）-docker&mysql
date: 2021-05-28 09:58:30
categories:
  - cicd
tags:
  - docker
  - linux
  - ubuntu
  - mysql
---

mysql也是测试项目中必用的，我们可以在docker中部署好mysql，用于我们在项目早期介入，便于灰盒测试。例如登录功能、详情功能等。

<!-- more -->

本文重新更新于2025-02-01

# 一、在宿主机（虚拟机）新建docker mysql数据挂载目录

```
$ mkdir -p ~/mysql/{conf,data,log}
```

# 二、运行docker mysql容器

```
$ sudo docker run \
-p 3306:3306 \
--restart=always \
--name test_mysql \
--privileged=true \
-v ~/mysql/log:/var/log/mysql \
-v ~/mysql/data:/var/lib/mysql \
-v ~/mysql/conf/my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=abc123456 \
-d mysql:8.3.0
```

# 三、检查docker容器

```
$ sudo docker ps
# 出现以下信息说明运行成功
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS         PORTS                                                  NAMES
5f32c38e2309   mysql:8.3.0   "docker-entrypoint.s…"   11 seconds ago   Up 9 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   test_mysql
```

# 四、进入docker的mysql容器，连接mysql服务器

```
$ sudo doccker exec -it 容器ID /bin/bash
$ mysql -u 用户名 -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.3.0 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

# 五、物理机（笔记本或台式电脑）测试连接虚拟机的3306端口

获取虚拟机的ip地址，在序号2：ens33中看到ip地址是192.168.52.131

虚拟机终端输入ip addr

```
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:bb:7c:8d brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.52.131/24 metric 100 brd 192.168.52.255 scope global dynamic ens33
       valid_lft 1024sec preferred_lft 1024sec
    inet6 fe80::20c:29ff:febb:7c8d/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:3e:5b:ed:29 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:3eff:fe5b:ed29/64 scope link
       valid_lft forever preferred_lft forever
5: vethb009301@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 66:1f:14:31:c4:92 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::641f:14ff:fe31:c492/64 scope link
       valid_lft forever preferred_lft forever
```

在物理机的navicat填好对应资料后，点击左下角的测试连接，可以看到连接成功

![](/pics/testing/navicat2.png)

双击左侧打开数据库

![](/pics/testing/navicat3.png)




参考资料：
https://blog.csdn.net/donkor_/article/details/139879575
https://mysql.net.cn/doc/refman/8.0/en/connecting-disconnecting.html
https://blog.csdn.net/Yvesty/article/details/119345796
