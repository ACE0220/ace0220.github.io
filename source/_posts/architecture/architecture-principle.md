---
title: 架构设计的三原则和应用
date: 2021-04-26 9:32:21
categories:
  - architecture
tags:
  - architecture
  - backend
  - frontend
  - 架构
  - 架构设计三原则
---

理解架构设计三原则和如何应用

<!-- more -->

## 核心内容

1. 理解架构设计三原则
2. 架构设计三原则的应用

## 三原则的介绍

- 简单原则
- 合适原则
- 演化原则

### 架构设计原则的目的、定位和作用

![](/pics/architecture/architecture-priciple-1.png)

原则的作用是指导做更好的设计，而不是可用的设计。

### 合适原则

业务复杂度，用户复杂度，很多架构一开始也不是适合的，是演进的来的。

设计出的架构要满足当时的业务需要，符合团队的能力水平

### 简单原则

**若无必要，勿增实体**

复杂度：分为内部复杂度和外部复杂度。

要先按照简单方式来设计架构，然后不断的迭代优化

内部复杂度和外部复杂度是天平两边，内部复杂度降低肯定会增加外部的复杂度，反之亦然。例如下图中的微服务体系，如果过度拆分，子服务内部的复杂度降低了，但是服务之间的链路复杂度就会增加，即外部复杂度增加。

![](/pics/architecture/architecture-priciple-2.png)

### 演化原则

演化优于一步到位，业务发展变化，要扩展，重构，甚至重写。

前期创造阶段是满足当前业务需求的一个架构，经过优化迭代演进，经过了重构，重写等方法，最终产出传承与适应变化的一个架构。


### 优先级

合适原则 > 简单原则 > 演化原则