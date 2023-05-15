---
title: 架构图的画法和架构方法论
date: 2021-04-10 19:58:30
categories:
  - architecture
tags:
  - architecture
  - backend
  - frontend
  - 架构
  - 架构图
---

画架构图中常用的工具和画法

<!-- more -->

## 4 + 1 架构视图

“4+1”视图是对逻辑架构进行描述，最早由 Philippe Kruchten 提出，他在1995年的《IEEE Software》上发表了题为《The 4+1 View Model of Architecture》的论文，引起了业界的极大关注，并最终被 [RUP (Rational Unified Process)](https://baike.baidu.com/item/%20RUP/8924595?fromModule=lemma_inlink) 采纳，现在已经成为架构设计的结构标准。

![](/pics/architecture/architecture-diagrams-1.jpeg)

- Logical View：逻辑视图，系统提供给用户的功能，对应uml的class和state diagrams
- Process View：处理视图，系统的处理过程，对应uml的sequece和activity diagrams
- Development View：开发视图，程序员角度的系统逻辑组成，对应uml的package diagrams
- Physical View：物理视图，系统工程师角度的物理组成，对应uml的deployment diagram
- Scenarios：场景，用户角度的系统需要实现的需求，对应uml的use case diagrams

一般来说，一个系统只有一个架构，但是从不同角度来看系统，可以得到不同的架构视角。

### 现状

较少企业使用4 + 1描述架构，因为

- 架构复杂度增加，现代基本是分布式系统
- 理解困难，4 + 1的逻辑视图，开发视图，处理视图比较容易混淆


## 常见架构图介绍和画法

### 架构图分类

![](/pics/architecture/architecture-diagrams-2.png)

客户端和前端都只需要按照模块划分，客户端和前端发布之后，一般不会像后端那样部署到多台服务，但是也会有部署到多台服务器的场景，特别是基于docker进行部署。

### 常见的架构图

alipay HK 简要业务架构

![](/pics/architecture/architecture-diagrams-3.png)


wechat 简要业务架构

![](/pics/architecture/architecture-diagrams-4.png)

mongodb 简要架构图

![](/pics/architecture/architecture-diagrams-5.png)



应用架构图， 每一个服务都是可以独立应用和部署

![](/pics/architecture/architecture-diagrams-6.png)

### 为什么后端逻辑架构直接叫系统架构

一般来说，后端要面临着更加多和复杂的功能需求，所以在架构上，前端有时更多也是配合后端，也并不说前端就没有架构一说，只是拿出了相对更复杂的一端来描述而已。

### 应用架构和系统架构的区别和联系

应用架构可以理解为是系统架构下一层的具体的一个细化，应用架构最主要能实现系统架构里面一些逻辑或者角色的功能。

## 系统序列图

4R中的运作规则Rule，一般都是通过系统序列图来展示。使用系统序列图来描述某个流程。

![](/pics/architecture/architecture-diagrams-7.png)

## 架构方法论

架构设计方法论常见的编程领域中，熟知的是面向对象和面向过程，而架构领域面向领域、面向风险、面向复杂度。

### 方法论的意义

### 面向模式

面向模式软件架构有五本书，POSA系列，是架构领域的设计模式。核心思想是经过验证的成熟架构模式，例如mvc，reator等。

- 面向模式的软件架构-模式系统
- 面向模式的软件架构-并发和联网对象模式
- 面向模式的软件架构-资源管理模式
- 面向模式的软件架构-分布式计算的模式语言
- 面向模式的软件架构-模式和模式语言

### 面向风险

风险驱动架构设计，核心思想是根据系统风险来设计软件架构，建模部分本质是面向对象设计的建模过程。

- 恰如其分的软件架构-风险驱动的设计方法
- 编程的逻辑

### DDD

DDD

- DDD是可扩展架构的设计技巧，不是架构方法论
- 兼顾架构和方案设计
- DDD、敏捷架构不关注存储和计算，只关注业务

books
- 领域驱动设计-阮籍啊核心复杂性对应之道
- 架构整洁之道

DDD为什么会抽象
- 兼顾架构设计和方案设计
- DDD的内容可以参考微服务的设计方案

### 面向复杂度的架构设计

面向复杂度方法论的核心

- 本质：为了降低软件系统的复杂度。
- 思路：通过分析系统需求找到系统复杂的地方，设计方案
- 模式：复杂度的来源是什么：高性能、高可用、可扩展、安全、成本。
- 套路：分库分表、缓存、集群、分片、微服务、DDD、异地多活

### 架构设计环


![](/pics/architecture/architecture-diagrams-8.png)

- 整体需求分析和判断
- 分析复杂度，就是4R中的role拆解
- 取舍，不同的架构方案，为什么要这么选
- 架构方案细化，4R架构设计
- 实现需求


### 为什么要做架构设计

- 提升开发效率，促进业务发展
- 公司流程要求
- 高性能、高可用、可扩展、可维护性等等。