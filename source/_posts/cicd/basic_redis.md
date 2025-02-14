---
title: 测试环境搭建（三）-docker&redis
date: 2021-06-03 09:58:30
categories:
  - cicd
tags:
  - docker
  - linux
  - ubuntu
  - mysql
  - redis
---

Redis 是一个强大的开源的高性能的key-value数据库，值 ( value ) 可以是字符串 ( String ) , 哈希 ( Map ) , 列表 ( list ) , 集合 ( Sets ) 或有序集合 ( Sorted Sets ) 等类型。每秒可以进行11万次读取和8万次写入操作。Redis还提供了多种特性，如发布/订阅、通知、key过期等。Redis采用自己实现的分离器来实现高速的读写操作，效率非常高。Redis是一个简单、高效、分布式、基于内存的缓存工具，通过网络连接提供Key-Value式的缓存服务。

<!-- more -->

# 一、获取docker redis镜像

唔~~~又到众所周知的原因了，想要拉取镜像，点击看[第一篇的3.3小节](/cicd/basic_docker/#3-3-docker源修改)

```
sudo docker pull redis:latest
```

```
$ sudo docker image ls
[sudo] password for ace:
REPOSITORY                  TAG       IMAGE ID       CREATED         SIZE
hub.zentao.net/app/zentao   latest    f0a33d3e597e   2 weeks ago     605MB
hello-world                 latest    74cc54e27dc4   3 weeks ago     10.1kB
redis                       latest    fa310398637f   5 weeks ago     117MB
mysql                       8.3.0     6f343283ab56   10 months ago   632MB
```

# 二、虚拟机创建映射目录，放置配置文件

```
# 路径由用户决定
$ cd ~
$ sudo mkdir -p redis/config
$ sudo mkdir -p redis/data
$ sudo mkdir -p redis/logs
# docker redis文档中提到以上的文件夹和文件都需要读写权限，那我偷个懒给与了全部读写执行权限
# -R 递归所有目录和文件
$ sudo chmod -R a+rwx redis
```

配置文件redis.conf，路径位于~/redis/config

```
aceserver:~/redis/config$ sudo vim redis.conf
```

```

配置文件
...
...
...
# 注释bind，让外部可以访问redis，或者可以指定ip访问，只是本地测试可以只注释就好
# bind 127.0.0.1 
# 用守护线程的方式启动,默认no
daemonize no
# 给redis设置密码
requirepass youpassword
# redis持久化，默认是no
appendonly no
# 远程主机检测与客户端的存活时间间隔 默认是300秒
tcp-keepalive 300
...
...
...
```

# 三、运行容器

```
$ sudo docker run -p 6379:6379 --restart=always --name redis_test -v ~/redis/config:/usr/local/etc/redis -v ~/redis/data:/data -v ~/redis/logs:/logs -d redis redis-server /usr/local/etc/redis/redis.conf
```

参数解释
-p 容器6379端口映射到宿主机6379
--name 容器name
-v \~/redis/config:/usr/local/etc/redis 容器内/usr/local/etc/redis文件夹映射到宿主机的~/redis/config，data和logs同理，这样redis.conf配置文件会存在于容器内/usr/local/etc/redis文件夹下
-d daemon守护进程运行

```
$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS                     PORTS                                                  NAMES
9a96660e8f33   redis         "docker-entrypoint.s…"   5 seconds ago   Exited (0) 3 seconds ago                                                          redis_test
```

检查容器运行发现退出了，检查一下

```
$ sudo docker logs 9a96660e8f33
# WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect
```

Redis 需要内存超分配（vm.overcommit_memory）被设置为 1。这意味着操作系统将始终允许分配所有请求的内存，这对于 Redis 的某些操作至关重要。
如果没有正确设置，当 Redis 尝试创建快照或进行某些类型的复制时，可能会因为内存不足而失败。这可能导致数据丢失或其他问题。

临时更改：在宿主机上执行以下命令,系统重启后失效

```
$ sudo sysctl vm.overcommit_memory=1
```

永久更改：编辑 /etc/sysctl.conf 文件

```
# 添加以下内容
vm.overcommit_memory = 1
```

运行

```
$ sudo sysctl -p
```


# 四、物理机（笔记本或台式电脑）测试连接虚拟机的6379端口

与mysql一样，使用navicat测试

在物理机的navicat填好对应资料后，点击左下角的测试连接，可以看到连接成功

![](/pics/testing/test_redis.png)

# 五、进入redis容器

进入redis容器并尝试登录和写入数据到数据库

```
$ sudo docker exec -it redis容器ID /bin/bash
# -a 代表password，但是这种方法并不安全，只是临时测试使用
$ redis-cli -a abc123456
127.0.0.1:6379> set mykey "hello"
OK
127.0.0.1:6379> get mykey
"hello"
127.0.0.1:6379>
```

物理机使用navicat查询数据

![](/pics/testing/test_redis_1.png)


参考资料：
https://blog.csdn.net/H1727548/article/details/132512038 介绍
https://developer.aliyun.com/article/1388047 docker部署
https://blog.csdn.net/qq_42146402/article/details/130204064 配置说明
https://hub.docker.com/_/redis 官方镜像
https://blog.csdn.net/m0_57236802/article/details/134726043 overcommit问题
https://redis.com.cn/commands.html 命令文档