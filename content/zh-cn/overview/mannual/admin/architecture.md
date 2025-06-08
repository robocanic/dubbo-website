---
aliases:
  - /zh/overview/reference/admin/architecture/
  - /zh-cn/overview/reference/admin/architecture/
description: ""
linkTitle: 架构
no_list: true
title: 架构
type: docs
weight: 2
---

回顾 [Dubbo 服务治理体系的总体架构](../../../what/overview/)，Admin定位于服务治理控制面中的一个核心组件，负责微服务集群的服务治理以及可视化展示。从这些角度出发，Admin不仅从宏观视角提供对整个数据面集群的观测数据，还能够提供一些基础的服务治理能力。

## 整体架构
![admin achitecture](/imgs/v3/reference/admin/console-new/admin-architecture.png)
如图，admin分为两部分：用户可见的前端控制台和相应的后端服务构成。后端服务集成了一系列可观测组件和服务发现/运行组件。其中，可观测组件都是以可插拔的形式集成进来，而对于服务发现组件如nacos/zookeeper等，admin内置了对这些服务发现方式的支持，同时admin也内置了对服务运行的基础设施Kubernetes的支持。


## RoadMap
未来，Dubbo Admin将围绕着如下功能进行演进：
1. 支持更多的部署场景
2. 完善服务治理的能力
3. 智能运维

如果你也对此有兴趣，欢迎贡献代码！
