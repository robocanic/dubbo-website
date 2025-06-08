---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/application/
  - /zh-cn/overview/reference/admin/tutorial/resources/application/
description: ""
linkTitle: 应用
no_list: true
title: 应用
type: docs
weight: 1
---
在Dubbo中，应用指的是一个独立部署的逻辑单元。通过一个应用，我们可以找到属于该应用的所有实例，这个应用提供的服务、消费的服务，属于该应用的路由规则等等。

在云原生场景中，会有deployment，statefulset等"工作负载"的概念，应用与工作负载的概念并不等同。在异地多活，多版本等场景下，一个应用可以有多个工作负载，它们管理的Pod属于同一个Dubbo应用。

## 详情
应用详情Tab是所属该应用所有实例信息的聚合展示，是所有实例的一个概况
![application-detail](/imgs/v3/reference/admin/console-new/application/application-detail.png)

## 实例
实例Tab展示了所属该应用的所有实例，可以快速根据ip，部署状态，启动时间，CPU，内存等字段筛选出相应的实例。
![application-instances](/imgs/v3/reference/admin/console-new/application/application-instances.png)

## 服务
服务Tab关联了该应用提供的服务以及该应用消费的服务，同样，可以根据服务名，QPS，RT等字段筛选出相应的服务
![application-services](/imgs/v3/reference/admin/console-new/application/application-services.png)


## 监控
dubbo admin集成了grafana dashboard，您可以通过嵌入的grafana iframe，来查看采集上来的监控指标。
![application-metric-dashboard](/imgs/v3/reference/admin/console-new/application/application-metric.png)
应用tab下的监控面板侧重于“应用”这一维度的指标聚合。与之对应的“实例”和“服务”则更侧重相应维度的指标聚合，您可以在[实例](./instance.md)和[服务](./service.md)的监控面板中看到更多的细节差异。

## 链路追踪
dubbo admin集成了grafana dashboard，您可以通过嵌入的grafana iframe，来查看采集上来的trace
![application-trace-dashboard](/imgs/v3/reference/admin/console-new/application/application-trace.png)


## 场景配置
场景配置是根据常见的场景，在对原子的路由规则进行封装后，开放给用户的便捷入口。就应用这个维度而言，Dubbo Admin封装了三种常见的配置，即访问日志，流量权重以及灰度隔离。

### 开始之前
1. [部署并打开shop商城项目](../quickstart.md)
2. [部署并打开Dubbo Admin](../quickstart.md)

### 访问日志
访问日志可以很好的记录某台机器在某段时间内处理的所有服务请求信息，包括请求接收时间、远端 IP、请求参数、响应结果等，运行态动态的开启访问日志对于排查问题非常有帮助。

商城的所有用户服务都由 User 应用的 UserService 提供，通过这个任务，我们为 User 应用的某一台或几台机器开启访问日志，以便观察用户服务的整体访问情况:
![application-accesslog](/imgs/v3/reference/admin/console-new/application/application-accesslog.png)

在新版Dubbo Admin中，如果你想要开启某个特定应用的执行日志，只需要在应用列表中选中应用，然后在配置页面打开执行日志的开关即可。
![application-accesslog-switch](/imgs/v3/reference/admin/console-new/application/application-accesslog-switch.png)

开启开关这一动作的背后其实是Dubbo Admin下发了一个如下的动态配置规则，其中accesslog: true就是开启访问日志的配置项
```yaml
configVersion: v3.0
enabled: true
key: shop-user
configs:
  - side: provider
    parameters:
      accesslog: true
```

### 流量权重
Dubbo 提供了基于权重的负载均衡算法，可以实现按比例的流量分布：权重高的提供者机器收到更多的请求流量，而权重低的机器收到相对更少的流量。

以基于权重的流量调度算法为基础，通过规则动态调整单个或一组机器的权重，可以在运行态改变请求流量的分布，实现动态的按比例的流量路由，这对于一些典型场景非常有用。
* 当某一组机器负载过高，通过动态调低权重可有效减少新请求流入，改善整体成功率的同时给高负载机器提供喘息之机。

* 刚刚发布的新版本服务，先通过赋予新版本低权重控制少量比例的流量进入，待验证运行稳定后恢复正常权重，并完全替换老版本。

* 服务多区域部署或非对等部署时，通过高、低权重的设置，控制不同部署区域的流量比例。


示例项目中，我们发布了 Order 服务 v2 版本，并在 v2 版本中优化了下单体验：用户订单创建完成后，显示订单收货地址信息。

![application-shop-order-v2](/imgs/v3/reference/admin/console-new/application/application-shop-order-v2.png)

现在如果你体验疯狂下单 (不停的点击 “Buy Now”)，会发现 v1 与 v2 总体上是 50% 概率出现，说明两者目前具有相同的默认权重。但我们为了保证商城系统整体稳定性，接下来会先控制引导 20% 流量到 v2 版本，80% 流量依然访问 v1 版本。

![application-shop-order-v2-structure](/imgs/v3/reference/admin/console-new/application/application-shop-order-v2-structure.png)
使用新版Dubbo Admin，选中shop-order应用后，选择配置Tab的的流量权重，配置实例v2的权重为25（Dubbo默认流量权重为100）：

![application-traffic-weight](/imgs/v3/reference/admin/console-new/application/application-traffic-weight.png)

再次疯狂点击 “Buy Now” 尝试多次创建订单，现在大概只有 20% 的机会看到 v2 版本的订单详情信息

在确定 v2 版本的 Order 服务稳定运行后，进一步的增加 v2 权重，直到所有老版本服务都被新版本替换掉，这样就完成了一次稳定的服务版本升级。

这个操作的背后是Dubbo Admin下发了一个动态配置规则：
```yaml
configVersion: v3.0
scope: application
key: shop-order
configs:
  - side: provider
    match:
      param:
        - key: orderVersion
          value:
            exact: v2
    parameters:
      weight: "25"
```

### 灰度隔离
无论是在日常开发测试环境，还是在预发生产环境，我们经常都会遇到流量隔离环境的需求。

* 在日常开发中，为了避免开发测试过程中互相干扰，我们有搭建多套独立测试环境的需求，但通过搭建物理集群的方式成本非常高且不够灵活

* 在生产发布过程中，为了保障新版本得到充分的验证，我们需要搭建一套完全隔离的线上灰度环境用来部署新版本服务，线上灰度环境能完全模拟生产运行情况，但只有固定的带有特定标记的线上流量会被导流到灰度环境，充分验证新版本的同时将线上变更风险降到最低。

以线上商城为例，我们决定为商城系统建立一套完整的线上灰度验证环境，灰度环境和线上环境共享一套物理集群，需要我们通过 Dubbo 标签路由从逻辑上完全隔离出一套环境，做到灰度流量和线上流量互不干扰。

![application-traffic-gray-structure](/imgs/v3/reference/admin/console-new/application/application-traffic-gray-structure.png)

使用新版Dubbo Admin，选中shop-user应用后，选择配置Tab的“灰度隔离”，点击“添加灰度环境”，指定env=gray的实例划入到gray这个灰度环境中：（其他应用操作类似）
![application-traffic-gray](/imgs/v3/reference/admin/console-new/application/application-traffic-gray.png)


在这个表单操作的背后，Dubbo Admin下发了一个标签路由规则：
```yaml
configVersion: v3.0
enabled: true
key: shop-user
tags:
- match:
  - key: env
    value:
      exact: gray
  name: gray

```
以上规则为每个应用隔离出了一套独立的灰度环境，所有带有 env=gray 的标签都属于灰度环境。等待一小会确保规则下发完成，接下来就可以验证灰度流量在隔离环境中运行。

为了模拟灰度流量，我们为商城示例首页设置了一个 Login To Gray 的入口来模拟从灰度环境进入商城的流量，在真实环境中这可以通过在入口网关根据某些规则识别流量并自动打标实现。
![shop-gray](/imgs/v3/tasks/gray/gray2.png)

通过 Login To Gray 登录后，之后所有请求 Detail、Comment、Order、User 服务的流量都会自动带有 dubbo.tag=gray 的标识，Dubbo 标签路由组件会识别这个标识，并将流量路由到刚才圈定的灰度环境（即所有 env=gray 的实例）。系统运行效果如下：
![shop-gray-3](/imgs/v3/tasks/gray/gray3.png)