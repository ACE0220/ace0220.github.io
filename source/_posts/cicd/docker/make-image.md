---
title: 构建自己的docker镜像
categories:
  - cicd
  - docker
tags:
  - microservice
  - node
  - docker
  - protobuffer
  - web
  - compose
date: 2021-05-01 21:53:00
---

在现有cicd的流程中，一般是通过Dockerfile去将生产代码绑定到镜像中，由于docker的沙箱能安全的隔离环境，更加能保证代码运行环境的一致性。

如果没了解或者没用过docker的，建议先看[一些入门教程](https://www.runoob.com/docker/docker-tutorial.html)或者看[官方文档，点击打开](https://docs.docker.com/get-started/)，因为本文不会很细致的讲一些安装、cli 命令之类的知识。主要还是提供一下构建镜像的流程，思路等。

<!-- more -->

# 本文需求

基于docker和grpc构建一个可以批量转换proto文件，生成typescript文件的一个镜像。

@tecace/ts-proto-batch: https://www.npmjs.com/package/@tecace/ts-proto-batch  github: https://github.com/ACE0220/ts-proto-batch

# Dockerfile

## Dockerfile是什么

Dockerfile 是用于构建镜像的配置文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

## Dockerfile部分指令简介

- FROM指令：用于指定我们的镜像基于哪个基础镜像，本文中的基础镜像是node（笔者的电脑是mac m1，所以用的基础镜像arm64v8/node）
- COPY指令: 从上下文目录中复制文件或目录到容器里指定路径。(上下文目录，即当前执行环境中相对或绝对目录，例如，在mac中执行构建镜像，那么上下文目录就是在mac中目录)
  - COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
  - [--chown=<user>:<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组
- RUN指令：用于执行后面跟着的命令行命令。有俩种格式
  - RUN <命令行命令> eg. RUN "node -v" 等价于在终端输入 node -v
  - RUN ["可执行文件", "参数1", "参数2"] eg. RUN ['node', '-v']
- CMD指令：与RUN指令类似，执行时机不同
  - RUN 运行在docker build阶段（构建过程中执行）
  - CMD 运行在docker run阶段（开始运行容器执行）
- VOLUME指令：定义要挂在的数据卷，在docker run的时候可以通过-v指定容器的数据卷挂载在宿主机的哪个目录，如果没有特别指定，会挂载在匿名卷中，避免数据丢失

# 基于debian镜像的安装与测试

先进行手动搭建等价于前期调研测试等，测试通过则可以将手动的步骤写入Dockerfile，通过Dockerfile生成镜像。

**笔者环境是mac m1，docker desktop, arm64v8/debian:stable-slim**

**笔者环境是mac m1，docker desktop, arm64v8/debian:stable-slim**

**笔者环境是mac m1，docker desktop, arm64v8/debian:stable-slim**


## 基于arm64v8/debian:stable-slim镜像运行container

运行容器

```sh
docker run -dit --name debian-test arm64v8/debian:stable-slim
```

进入容器内部，并安装基础依赖

```sh
docker exec -it debian-test bash
```

```sh
apt update
apt install sudo vim wget curl unzip
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

## protoc 安装

Protobuf即Protocol Buffers，是Google公司开发的一种跨语言和平台的序列化数据结构的方式，是一个灵活的、高效的用于序列化数据的协议。

而protoc则是官方提供的一种编译器，用于编译proto文件，可以编译成java、python、go等代码

下载protoc二进制包

```sh
wget https://github.com/protocolbuffers/protobuf/releases/download/v23.1/protoc-23.1-linux-aarch_64.zip
```

解压

```sh
unzip protoc-23.1-linux-aarch_64.zip
```

复制文件

```sh
cp bin/protoc /usr/local/bin/
cp -r include/google/ /usr/local/include/
```

测试，终端输入protoc

```sh
Usage: protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
                              If not found in any of the these directories,
                              the --descriptor_set_in descriptors will be
                              checked for required proto file.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
```

## npm相关的包安装ts-proto @tecace/ts-proto-batch

ts-proto是一个protoc的插件，经过插件处理后的数据可以生成ts文件，@tecace/ts-proto-batch是笔者基于ts-proto开发的一个小模块。

功能主要是读取批量的protobuf文件，并按照输入的目录结构一一对应地生成相应的.ts文件。

@tecace/ts-proto-batch地址：https://www.npmjs.com/package/@tecace/ts-proto-batch

```sh
npm install -g ts-proto @tecace/ts-proto-batch
```

## proto转换ts文件测试

创建测试用的文件和文件夹

```sh
cd ~ 
mkdir tspb && cd tspb 
touch top.proto 
mkdir sub && cd sub 
touch sub.proto
```

top.proto 和 sub.proto填入一下内容

top.proto
```text
syntax = "proto3";

package demo;

message Top {
    int64 client_id = 1;
    string request_data = 2;
}
```

sub.proto
```text
syntax = "proto3";

package demo;

message Sub {
    int64 client_id = 1;
    string request_data = 2;
}
```

转换测试，执行一下命令后打开dist文件夹，里面会按照-i选项的目录结构生成对应的ts文件

```sh
ts-pb gen -i . -o dist
```

# 基于以上流程构建docker镜像

## 思路

把前文中所有在debian镜像中的搭建流程融入shell脚本文件和Dockerfile。

前置的准备要不少，例如安装好node环境，protoc编译器，安装需要的npm包等，前置工作就可以放在shell脚本中。

那么Dockerfile可以做什么呢？

Dockerfile的主要作用是在脚本运行之前做一些准备工作，脚本中的环境配置结束之后，运行命令。

## shell脚本

```sh
#! /usr/bin/env sh

cd ~
mkdir data && cd data
apt update
echo y | apt install sudo vim wget curl unzip
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v

wget https://github.com/protocolbuffers/protobuf/releases/download/v23.1/protoc-23.1-linux-aarch_64.zip
unzip protoc-23.1-linux-aarch_64.zip
cp bin/protoc /usr/local/bin/
cp -r include/google/ /usr/local/include/
npm install -g ts-proto @tecace/ts-proto-batch
```

## Dockerfile

```sh
FROM arm64v8/debian:stable-slim

# 环境准备工作，通过创建文件夹，给文件和文件夹赋予权限，然后运行脚本等。
RUN mkdir /home/env && chmod -R 777 /home/env
COPY docker-build.sh /home/env
RUN chmod -R 777 /home/env/docker-build.sh && /home/env/docker-build.sh
RUN mkdir /home/data
WORKDIR /home/data
VOLUME [ "/home/data" ]
# 在环境准备好之后，运行proto转换程序
CMD ts-pb gen -i protos -o dist
```

## 开始构建镜像

Dockerfile和docker-build.sh位于同一个文件夹下，运行以下命令

**注意，命令最后还有一个.   这个符号代表docker build的时候的base path，在docker file中，copy文件的路径是基于命令最后的base path**

例如： 

base path 是 . , dockerfile中的 COPY docker-build.sh /home/env，docker-build.sh会被处理成 "./docker-build.sh",

如果是base path是 ../docker-shell，docker-build.sh会被处理成 "../docker-shell/docker-build.sh"

```sh
docker build -t arm64v8/ts-proto-batch:v0.0.1 .
```

## 启动构建好的镜像

在镜像中固定为挂载路径下的protos存放proto 文件，生成的dist文件夹在protos文件夹同级

所以 -v 参数一定要提供，不提供无法识别到proto文件在哪个文件夹

```sh
docker run -dit --name tspb -v /Users/ace/volumes/tspb:/home/data arm64v8/ts-proto-batch:v0.0.1
```

- /Users/ace/volumes/tspb 这个路径一定要通过 -v 参数提供，由用户进行自定义
  - /Users/ace/volumes/tspb/protos 这里放置proto文件，支持多级文件夹
  - /Users/ace/volumes/tspb/dist 运行结束会自动生成，dist文件夹内部结构与protos一致





