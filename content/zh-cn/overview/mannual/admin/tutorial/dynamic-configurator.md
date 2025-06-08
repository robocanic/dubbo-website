---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/dynamic-configurator/
  - /zh-cn/overview/reference/admin/tutorial/resources/dynamic-configurator/
description: ""
linkTitle: 动态配置
no_list: true
title: 动态配置
type: docs
weight: 6
---

## 规则说明

动态配置规则 (ConfigurationRule) 是 Dubbo 设计的在无需重启应用的情况下，动态调整 RPC 调用行为的一种能力，也称为动态覆盖规则，因为它是通过在运行态覆盖 Dubbo 实例或者 Dubbo 实例中 URL 地址的各种参数值，实现改变 RPC 调用行为的能力。动态配置可以作用于应用或者服务。

同样，Dubbo Admin对动态配置也进行了表单化。下面以一个具体的例子来展示如何借助dubbo admin的表单操作来下发动态配置规则。

## 新增动态配置规则
```yaml
key: org.apache.dubbo.samples.UserService
scope: service
configVersion: v3.0
enabled: true
configs:
  - side: consumer
    parameters:
      retries: 5
```

如上的动态配置规则含义是：`org.apache.dubbo.samples.UserService`这个服务的所有消费者对该接口的调用重试次数配置为5（默认为2）。

以下是使用Dubbo Admin前端的操作过程：
1.  在流量管控这个一级菜单中选择“动态配置”，点击“新增动态配置”
  ![dynamic-configurator-list](/imgs/v3/reference/admin/console-new/dynamic-configurator/dynamic-configurator-list.png)
2. 对表单进行操作，填写好规则的基本信息与配置项
  ![dynamic-configurator-form](/imgs/v3/reference/admin/console-new/dynamic-configurator/dynamic-configurator-form.png)
3.  点击保存后，可以验证这个规则是否下发成功
4.  新增的规则，可以在列表中看到，并可以对它进行查看/修改/删除操作

## 删除动态配置规则
返回规则列表，在列表中找到要删除的指定规则，点击规则，删除即可：
![delte](/imgs/v3/reference/admin/console-new/dynamic-configurator/delete.png)