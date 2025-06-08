---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/condition-rule/
  - /zh-cn/overview/reference/admin/tutorial/resources/condition-rule/
description: ""
linkTitle: 条件路由
no_list: true
title: 条件路由
type: docs
weight: 4
---

## 规则说明

条件路由规则将符合特定条件的请求转发到特定的地址实例子集上。规则首先对发起流量的请求参数进行匹配，符合匹配条件的请求将被转发到包含特定实例地址列表的子集。

在Dubbo Admin前端中，为了降低这些条件路由的使用门槛，对条件路由规则进行了表单化，您可以直接操作表单即可完成规则的填充和下发。除此之外，很多场景配置背后其实是Admin封装的条件路由规则。

条件路由规则表单化后，主要分为**请求匹配**和**路由分发**两部分。请求匹配即一个condition中"=>"之前的内容，路由分发即"=>"之后的内容。

## 新增条件路由规则

以下是一个条件路由规则示例：

```yaml
configVersion: v3.0
scope: service
force: true
runtime: true
enabled: true
key: org.apache.dubbo.samples.CommentService
conditions:
  - method=getComment => region=Hangzhou
```

上面规则的含义是：所有 `org.apache.dubbo.samples.CommentService` 服务 `getComment` 方法的调用都将被转发到有 `region=Hangzhou` 标记的地址子集。

以下是使用Dubbo Admin前端的操作过程：
1. 在流量管控这个一级菜单中选择“条件路由”，点击“新增条件路由规则”
2. 填充规则的metadata，即基本信息
  ![condition-rule-metadata](/imgs/v3/reference/admin/console-new/condition-rule/condition-rule-metadata.png)
3. 在路由列表中/新增一条路由，并按照上述规则体进行表单操作：
  ![condition-rule-form](/imgs/v3/reference/admin/console-new/condition-rule/condition-rule-form.png)
4. 点击Yaml视图，可以看到表单视图中的操作修改同步到了yaml视图
5. 确认新增后，可以验证这个规则是否下发成功


## 删除条件路由规则
在规则列表中，找到条件路由规则，在操作列点击删除即可：
![condition-rule-form](/imgs/v3/reference/admin/console-new/condition-rule/delete.png)
