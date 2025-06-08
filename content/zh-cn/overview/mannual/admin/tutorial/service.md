---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/sevice/
  - /zh-cn/overview/reference/admin/tutorial/resources/service/
description: ""
linkTitle: 服务
no_list: true
title: 服务
type: docs
weight: 3
---

在 Dubbo 中，​服务​​是业务逻辑的最小协作单元（如一个 Java 接口），通过注册中心暴露接口定义与调用规则，直接参与服务注册、发现及治理。而在云原生场景中，​Kubernetes Service​​ 负责为容器化应用（如 Dubbo 服务实例）提供统一访问入口和负载均衡。两者的核心差异在于：​Dubbo 服务是业务逻辑的协作单元，聚焦接口定义与远程调用治理；而 K8s Service 是基础设施的网络抽象单元，聚焦流量分发与容器化资源的路由管理​​。如果没有特殊说明，Dubbo Admin中的服务（Service）指的就是第一种：[RPC接口集合](https://opensergo.io/zh-cn/docs/what-is-opensergo/concepts/)

## 分布
在服务分布 Tab 可以查看当前服务对应的生产者与消费者应用和实例的信息，包括应用名、实例数、地址、调用配置等元数据。
![service-distribution](/imgs/v3/reference/admin/console-new/service/service-distribution.png)


## 监控
服务下的监控是以"service"这一维度进行聚合的，你可以通过看到该服务所有接口的详细调用指标。
![service-metric-dashboard](/imgs/v3/reference/admin/console-new/service/service-metric.png)

## 链路追踪
同样，链路追踪展示的也是"服务"这一维度的，你可以筛选出最近调用该服务的trace，并进行进一步查看。
![service-trace-dashboard](/imgs/v3/reference/admin/console-new/service/service-trace.png)

## 场景配置

### 开始之前
1. [部署并打开shop商城项目](../quickstart.md)
2. [部署并打开Dubbo Admin](../quickstart.md)



### 超时时间
Dubbo 提供动态调整服务超时时间的能力，在无需重启应用的情况下调整服务的超时时间，这对于临时解决一些服务上下游依赖不稳定而导致的调用失败问题非常有效
商城项目通过`org.apache.dubbo.samples.UserService`提供用户信息管理服务，访问 http://localhost:8080/ 打开商城并输入任意账号密码，点击 Login 即可以正常登录到系统。
![timeout1](/imgs/v3/tasks/timeout/timeout1.png)
有些场景下，User 服务的运行速度会变慢，比如存储用户数据的数据库负载过高导致查询变慢，这时就会出现 UserService 访问超时的情况，导致登录失败。
![timeout2](/imgs/v3/tasks/timeout/timeout2.png)

在示例系统中，可通过下图 Timeout Login 模拟突发的 UserService 访问超时异常
![timeout4](/imgs/v3/tasks/timeout/timeout4.png)

为了解决突发的登录超时问题，我们只需要适当增加 UserService 服务调用的等待时间即可。
![timeout3](/imgs/v3/tasks/timeout/timeout3.png)

在新版Dubbo Admin的控制台中，可以通过如下两种途径来调整超时时间。

1. 在动态配置原子规则的配置中，新增如下配置规则。
  ```yaml
  configVersion: v3.0
  enabled: true
  configs:
    - side: provider
      parameters:
        timeout: 2000
  ```
2. 找到`org.apache.dubbo.samples.UserService`的这个服务，在场景配置中，配置对应的超时时间。事实上，场景配置背后也是转化成了同样的动态配置规则下发到dubbo服务的。
![service-timeout](/imgs/v3/reference/admin/console-new/service/service-timeout.png)

保存后，尝试多次刷新页面，发现用户详情数据总是能正常显示，虽然有时由于重试的缘故加载时间会明显变长。

### 重试次数

重试次数用于定义消费者在调用失败后的自动重试次数​​，表示若首次调用失败，消费者会最多尝试 ​n 次重试，若仍失败则抛出异常。
在服务初次调用失败后，通过重试能有效的提升总体调用成功率。但也要注意重试可能带来的响应时间增长，系统负载升高等，另外，重试一般适用于只读服务，或者具有幂等性保证的写服务。

成功登录商城项目后，商城会默认在首页展示当前登录用户的详细信息。
![retry1](/imgs/v3/tasks/retry/retry1.png)
但有些时候，提供用户详情的 Dubbo 服务也会由于网络不稳定等各种原因变的不稳定，比如我们提供用户详情的 User 服务就很大概率会调用失败，导致用户无法看到账户的详细信息。用户账户详情查询失败后的系统界面如下：
![retry4](/imgs/v3/tasks/retry/retry4.png)
商城为了获得带来更好的使用体验，用户信息的加载过程是异步的，因此用户信息加载失败并不会影响对整个商城页面的正常访问，但如果能始终展示完整的用户信息总能给使用者留下更好的印象。
考虑到访问用户详情的过程是异步的（隐藏在页面加载背后），只要最终数据能加载出来，适当的增加等待时间并不是大的问题。因此，我们可以考虑通过对每次用户访问增加重试次数的方式，提高服务详情服务的整体访问成功率。
![retry3](/imgs/v3/tasks/retry/retry3.png)

同样，我们有两种途径来动态调整重试次数
1. 在动态配置中，新增如下规则:
  ```yaml
    configVersion: v3.0
    enabled: true
    configs:
      - side: consumer
        parameters:
          retries: 5
  ```
2. 找到`org.apache.dubbo.samples.UserService`的服务，在场景配置中，配置重试次数为5即可。Admin背后会转化成同样的动态配置规则下发到Dubbo 服务。
  ![service-retry](/imgs/v3/reference/admin/console-new/service/service-retry.png)

### 同区域优先
为了保证服务的整体高可用，我们经常会采用把服务部署在多个可用区(机房)的策略，通过这样的冗余/容灾部署模式，当一个区域出现故障的时候，我们仍可以保证服务整体的可用性。

当应用部署在多个不同机房/区域的时候，应用之间相互调用就会出现跨区域的情况，而跨区域调用会增加响应时间，影响用户体验。同机房/区域优先是指应用调用服务时，优先调用同机房/区域的服务提供者，避免了跨区域带来的网络延时，从而减少了调用的响应时间


Detail 应用和 Comment 应用都有双区域部署，其中 Detail v1 与 Comment v1 部署在区域 Beijing，Detail v2 与 Comment v2 部署在区域 Hangzhou 区域。为了保证服务调用的响应速度，我们需要增加同区域优先的调用规则，确保 Beijing 区域内的 Detail v1 始终默认调用 Comment v1，Hangzhou 区域内的 Detail v2 始终调用 Comment v2。
![region1](/imgs/v3/tasks/region/region1.png)
当同区域内的服务出现故障或不可用时，则允许跨区域调用。
![region2](/imgs/v3/tasks/region/region2.png)

正常登录商城系统后，首页默认展示商品详情信息，多次刷新页面，发现商品详情 (description) 与评论 (comment) 选项会出现多个不同版本的组合，结合上面 Detail 和 Comment 的部署结构，这说明服务调用并没有遵循同区域优先的原则。
![region3](/imgs/v3/tasks/region/region3.png)

因此，接下来我们需要添加同区域优先规则，保证：

hangzhou 区域的 Detail 服务调用同区域的 Comment 服务，即 description v1 与 comment v1 始终组合展示
beijing 区域的 Detail 服务调用同区域的 Comment 服务，即 description v2 与 comment v2 始终组合展示

在新版admin console上，找到对应服务的场景配置也，一键开启同区域优先即可
![same-region-first](/imgs/v3/reference/admin/console-new/service/same-region-first.png)

同区域优先开启后，此时再尝试刷新商品详情页面，可以看到 description 与 comment 始终保持 v1 或 v2 的同步。
![region4](/imgs/v3/tasks/region/region4.png)

这个开关对应admin下发的规则如下：
```yml
configVersion: v3.0
enabled: true
force: false
key: org.apache.dubbo.samples.CommentService
conditions:
  - '=> region = $region'
```

