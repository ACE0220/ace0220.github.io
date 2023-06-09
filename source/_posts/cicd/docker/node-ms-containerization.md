---
title: node 微服务简介与容器化入门
categories:
  - microservice
tags:
  - microservice
  - node
  - docker
  - nest.js
date: 2021-05-02 18:09:00
---

微服务，采用容器化技术，将大型系统（服务）拆分为多个子系统（服务）的一种架构方案。服务之间采用明确的api进行通信，服务之间可以独立部署和维护。

<!-- more -->

# 微服务基本介绍

## 微服务是什么，为什么需要微服务

是什么

- 微服务，采用容器化技术，将大型系统（服务）拆分为多个子系统（服务）的一种架构方案。服务之间采用明确的api进行通信，服务之间可以独立部署和维护。
- 微服务里面的服务仅用于一个特定业务功能

为什么

- 微服务的逻辑更加清晰
- 微服务能更好的快速迭代
- 团队协作更加方便清洗，多语言可以灵活组合

## 微服务中的DDD是什么

领域驱动设计，Domain Driven Design

- 在微服务开发过程中，粒度的划分是一个难点，怎么合理划分微服务，在划分过程中，就会需要用到DDD。
- 康威定律：设计系统的架构受制于产生这些设计的组织的沟通结构

DDD的作用-决定软件复杂性的是**设计方法**

- DDD有助于指导确定系统边界
- 能够聚焦于系统核心元素
- 帮助拆分系统

## DDD的常用概念-领域

- 领域：领域是有范围界限的，可以说是有边界
- 核心域：是业务系统的核心价值
- 通用子域：所有子域的消费者，提供通用服务
- 支撑子域：专注于业务系统某一个重要的业务

例如：
- 电商是一个领域
- 子域有商品子域，用户子域，订单子域，销售子域等
- 核心域可能是销售子域（或者说暂定销售子域是核心）。
- 用户子域，商品子域等定位如果是为了支撑销售，那么他们可以是支撑子域
- 通用子域可以是数据操作，第三方服务（通知，短信，监控）等。

## 界限上下文

- 边界：用于描述一个系统中的分离和聚合点，将一个大型系统分为多个不同的业务子域，每个子域内都包含其自身的数据模型、业务规则和流程等等，并拥有其特定的边界。

- 界限上下文：包括一系列实体、值对象、聚合、工厂、存储库和服务等，界限上下文内部把相关的实体、值对象、聚合、工厂、存储库和服务等组织在一起

- 目的：不在于如何划分边界，在于如何控制边界

例如：

商品管理子域的边界：

- 商品属性和分类管理
- 商品上下架管理
- 商品库存管理

商品管理子域的界限上下文：

- 商品的实体：价格，属性，分类等
- 商品的操作：定义商品的上架和下架、增删改查等操作
- 库存的操作：存储商品或者操作商品的库存

## 领域模型

是什么

- 理解：对软件系统中要解决问题的抽象表达
- 领域：反映的是业务上要解决的问题
- 模型：针对问题提出的解决方案

## DDD四层架构

经典四层微服务架构

![](/pics/microservice/node-ms-containerization-1.png)

- interface 用户展示
- Application 协调工作
- Domain 实现业务规则
- Infrastructure 中间件，mysql，云设施等

详细拆分

![](/pics/microservice/node-ms-containerization-2.png)

## 微服务的设计原则

- 领域驱动设计，不是数据驱动或界面驱动设计
- 边界清晰的微服务，而不是非常小的单体
- 职能清晰的分层，避免过度拆分微服务

# docker快速入门和使用

## docker

为什么

- 软件更新发布和部署抵消，过程繁琐，人工介入
- 环境一致性难以保证，不同环境迁移成本太高
- 构建容易，分发简单

应用场景

- 构建运行环境
- 微服务
- CI/CD环境的高度一致性

## docker安装

- 物理机 mac m1
- 虚拟机 paralles ubuntu server arm64 linux

官方文档：https://docs.docker.com/engine/install/ubuntu/

升级apt包的索引，以允许通过https安装包

```sh
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

添加docker官方GPG key

```sh
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

使用一下命令设置存储库

```sh
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安装docker engine, containerd, docker compose

```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

测试安装是否成功

```sh
sudo docker run hello-world
```

终端输出

```sh
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
70f5ac315c5a: Pull complete 
Digest: sha256:fc6cf906cbfa013e80938cdf0bb199fbdbb86d6e3e013783e5a766f50f5dbce0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!

```

## docker的相关概念

- 客户端client：可运行docker指令
- 服务进程docker daemon：管理镜像和容器
- 镜像仓库regisry：存储镜像的仓库

## docker基本操作

- 仓库操作：docker pull, docker push
- 镜像管理：docker images, rmi, build
- 生命周期管理：docker run, start, stop, rm


# gRPC 和 ProtoBuf

## RPC和gRPC介绍

RPC是什么

- RPC：远程过程调用remote procedure call
- 包含了传输协议和编码协议
- 允许运行于一台计算机的程序调用另一台计算机的子程序

gRPC

- 是一个高性能，开源，通用rpc框架

- 基于http2.0协议标准设计开发

- 支持多语言，采用protocal buffers数据序列化协议

gRPC调用的流程

![](/pics/microservice/node-ms-containerization-3.png)

- 客户端程序发送函数请求RPC call
  - 在client stub中把要请求的参数进行序列化，通过协议进行编码
  - 编码后的数据通过网络发送请求
- 服务端接收到数据
  - 通过server stub中的协议编码进行解码，然后进行反序列化
  - 调用对应的服务端程序，返回结果

## ProtoBuf及详细语法介绍

**是什么**

- 高效的序列化结构化数据的协议
- 通常用在存储数据和需要远程数据通信的程序上
- 跨语言，更加轻便

**为什么**

- 加速数据传输效率
- 解决数据传输不规范的问题


**常用的概念**

- message定义：描述了一个请求或者响应的消息格式
- 字段表示：消息的定义中，每一个字段都有唯一的数值标签
- 常用数据类型：double, float, int32/64, bool, string, bytes
- service服务定义，在service中可以定义一个rpc服务接口

**protocol buffers message中字段修饰符**

- singular：表示成员有0个或者一个，一般可以省略
- repeated： 表示可以拥有0～N个元素

**protobuf类型**

官方文档：https://protobuf.dev/programming-guides/proto3/#scalar

**第一个protobuf文件**

```text
# 版本号
syntax = 'proto3';

# 包名
package node.micro.service.product;

# 服务
service Product {
  rpc add(ProductInfo) return (ResponseProduct) {}
}

# 请求的message
message ProductInfo {
  int32 id = 1;
  string name = 2;
}

# 返回的message
message ResponseProduct {
  int32 id = 1;
}
```

# 搭建自己的doker镜像

https://ace0220.github.io/cicd/docker/make-image/

# nest.js 微服务基本入门

https://ace0220.github.io/microservice/intro-nest/

