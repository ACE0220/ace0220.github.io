---
title: 测试环境搭建（一）-docker&ubuntu
date: 2021-05-27 09:58:30
categories:
  - cicd
tags:
  - docker
  - linux
  - ubuntu
---

为了测试环境的一致性，可以先从docker搭建开始，docker是所有必要工具的搭建基础，虽然在大部分成熟的企业中已经做好了相应配置，但是了解搭建过程和目的对于个人成长也是很必要的。

<!-- more -->

本文重新更新于2025-02-01

# 一、物理机、虚拟机和操作系统

本地采用vmware + ubuntu24.04搭建虚拟机，物理机是x86_64，windows11

# 二、虚拟机环境搭建的注意点

安装参考这个网址：https://blog.csdn.net/m0_70885101/article/details/137694608
再自己摸索一下问题不大

## 换源&apt update


由于众所周知的原因，ubuntu的apt源大概率要更换成**国内镜像**

apt源的配置文件已经更换到 **/etc/apt/sources.list.d/ubuntu.sources**

源文件

```
$ cd /etc/apt/sources.list.d/
$ cat ubuntu.sources
```

```

Types: deb
URIs: http://cn.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

备份源文件

```
$ cp ubuntu.sources ubuntu.sources.bak
```

修改ubuntu.sources

```
$sudo vim ubuntu.sources

# 修改成以下内容

Types: deb
URIs: https://mirrors.aliyun.com/ubuntu/
Suites: noble noble-updates noble-backports noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 修改完成，退出insert模式，:wq保存和关闭文件
```

apt更新

```
$ sudo apt update
```

# 三、docker engine安装

## 3.1 卸载旧版本

linux可能提供非官方的docker包，与官方的可能会冲突，必须先卸载一下非官方软件包

- docker.io
- docker-compose
- docker-compose-v2
- docker-doc
- podman-docker

```
$ for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

镜像、容器、网络等存储在/var/lib/docker/，删除docker的时候不会移除。如果需要干净的安装环境，选择先清除已存在的数据，参考这个链接https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine

## 3.2 使用apt进行安装

1.设置docker的apt存储库

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2.安装docker的包

```
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 3.3 docker源修改

又因为众所周知的原因，docker也需要修改为国内源

```
$ cd /etc/docker/
```

```
$ sudo vim daemon.json
```
填入以下内容
```
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.xuanyuan.me"
    ]
}
```

重启docker

```
systemctl daemon-reload
systemctl restart docker
```

测试国内源

```
$ sudo docker run hello-world
# 出现以下信息证明测试成功了
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete
Digest: sha256:d715f14f9eca81473d9112df50457893aa4d099adeb4729f679006bf5ea12407
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

本文重新更新于2025-01