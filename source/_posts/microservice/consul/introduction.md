---
title: 注册中心consul
categories:
  - microservice
  - consul
tags:
  - microservice
  - cicd
  - service mesh
  - consul

date: 2021-06-17 16:53:00
---

Consul是一种服务网格解决方案，提供具有服务发现，配置和分段等功能。

<!-- more -->

# 注册中心consul

## consul的基本介绍

Consul是一种服务网格解决方案，提供了服务发现，配置和分段功能的全功能控制平台，内置代理，支持开箱即用。

## 关键功能

- 服务发现：客户端可以注册服务，程序可以轻松找到他们所依赖的服务。
- 运行状况检查：consul客户端可以提供任意数量的运行状态检查。
- KV存储：程序将consul的层级key/value存储用于任何目的，包括动态配置，功能标记，协调，领导选举等。
- 安全的服务通信：可以为服务生成和分发生成TLS证书
- 多数据中心：支持多个数据，在不同云平台建立集群，可以完美融合，避免其中一个平台的consul服务器宕机导致全站挂掉。

## 多数据中心 

![](/pics/cicd/consul/intro-1.png)

## 架构和协议

上图中有两个数据中心，数据中心之间通过WAN GOSSIP协议通信

CLIENT和SERVER之间通过RPC通信，每个数据中心有一个Leader Server

CLIENT之间通过LAN GOSSIP协议进行通信，进行数据的同步

### 注册中心的重要协议

- Gossip Protocol 八卦协议
  - 八卦协议，作用如其名，打听其他节点发生了什么事，方便于自身去同步
- Raft Protocol 选举协议
  - 用于选举出一个leader节点

#### gossip协议

局域网池LAN Pool

- 让client自动发现server节点，减少所需要的配置量
- 分布式故障检测在某几个server机上执行
- 能够快速的广播事件

广域网池WAN Pool

- WAN Pool是全局唯一的
- 不同数据中心server都会加入WAN Pool
- 允许服务器执行夸数据中心请求

## 注册中心consul的主要特性

- 服务发现
- 健康检查
- 键值对存储

### 注册中心的访问过程

![](/pics/cicd/consul/intro-2.png)

# 注册中心的安装

```sh
# m1
docker pull arm64v8/consul:latest
# other
docker pull consul:latest
```

```sh
docker run -dit --name consul-test -p 8500:8500 arm64v8/consul:latest
```

浏览器打开localhost:8500

- services 服务，consul本身也计算进服务内
- Nodes 当前所有的节点和健康状态
- key/value 键值对存储
- intentions 用于控制服务之间通信流量的一种机制，其作用是限制哪些服务能够相互通信

![](/pics/cicd/consul/intro-3.png)

