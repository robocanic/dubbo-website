---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/instance/
  - /zh-cn/overview/reference/admin/tutorial/resources/instance/
description: ""
linkTitle: 实例
no_list: true
title: 实例
type: docs
weight: 2
---

在 Dubbo 中，实例是服务提供者或消费者的最小运行单元（如一个 JVM 进程），通过注册中心暴露自身的服务地址与接口信息，直接参与服务调用与治理。而在云原生场景中，​Pod 是资源调度的最小原子单位，负责承载容器化进程（如 Dubbo 实例）的运行环境（网络、存储等）。两者的核心差异在于：​Dubbo 实例是业务逻辑的服务单元，聚焦服务暴露与调用，而Pod 是基础设施的资源单元，聚焦容器生命周期管理。在Admin Console中展示的实例，则是两者之间的结合。


## 详情
实例详情Tab是所属该工作负载下具体实例的详细信息展示，涵盖实例IP、端口、部署状态、注册状态及所属应用等元数据
![instance-detail](/imgs/v3/reference/admin/console-new/instance/instance-detail.png)

## 监控
实例嵌入的grafana监控面板如下，该页面看到的指标是以“实例”这个维度进行聚合的
![instance-metric-dashboard](/imgs/v3/reference/admin/console-new/instance/instance-metric.png)
## 链路追踪
实例的trace如下所示
![instance-trace-dashboard](/imgs/v3/reference/admin/console-new/instance/instance-trace.png)
## 场景配置

### 访问日志
实例的访问日志Tab与应用的访问日志功能相同，区别在于实例的 tab 可以单独指定当前实例开启访问日志。
![instance-accesslog](/imgs/v3/reference/admin/console-new/instance/instance-accesslog.png)

开启后Dubbo Admin会下发一个动态配置规则，其中`{ip}` 为具体的实例地址。

```yaml
configVersion: v3.0
enabled: true
configs:
  - match
     address:
       oneof:
        - wildcard: "{ip}:*"
    side: provider
    parameters:
      accesslog: true
```
### 流量禁用 
在 Dubbo 实例出现异常、需要临时下线维护或有安全风险时，通过​**​流量禁用功能​**​可立即切断该实例的请求接收，避免故障扩散或资源耗尽，同时通过页面底部的​**​提交​**​按钮使配置即时生效，保障服务整体稳定性。
![instance-traffic-disable.png](/imgs/v3/reference/admin/console-new/instance/instance-traffic-disable.png)

> 由于流量禁用背后的动态配置规则不会随着实例下线而删除，在排查完问题后，需要手动恢复实例流量！