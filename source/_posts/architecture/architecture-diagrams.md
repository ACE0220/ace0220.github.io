---
title: 如何画架构图
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