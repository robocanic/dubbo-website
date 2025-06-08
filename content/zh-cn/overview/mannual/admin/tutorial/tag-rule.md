---
aliases:
  - /zh/overview/reference/admin/tutorial/resources/tag-rule/
  - /zh-cn/overview/reference/admin/tutorial/resources/tag-rule/
description: ""
linkTitle: 标签路由
no_list: true
title: 标签路由
type: docs
weight: 5
---
## 规则说明

标签路由（Tag Routing）是Dubbo提供的流量隔离能力，通过为实例打标（标签）并将请求按规则路由到特定标签组，实现**逻辑环境隔离**和**精细化流量控制**。核心机制：

* **实例打标**​：为实例分配标签（如`env=gray`），形成逻辑分组。

* **请求匹配**​：根据请求上下文（参数、Header等）匹配标签规则，路由到对应实例组。

## 新增标签路由规则
以下是一个标签路由规则示例：

```yaml
configVersion: v3.0
 force: true
 enabled: true
 key: shop-detail
 scope: application  
 tags:
   - name: gray
     match:
       - key: version
         value:
           exact: v1
```

上面规则的含义是：所有 `shop-detail` 应用中，元数据 `version` 精确为 `v1` 的服务提供者实例，会被动态标记为 `gray` 标签。消费者若携带 `gray` 标签（如通过 `RpcContext` 或请求头），其流量将**仅路由到标记为gray的实例**，实现灰度环境隔离。

以下是使用Dubbo Admin前端的操作过程：

1. 在流量管控这个一级菜单中选择“标签路由”，点击“新增标签路由规则”
2. 填充规则的metadata，即基本信息
  ![meta](/imgs/v3/reference/admin/console-new/tag-rule/meta.png)
3. 在路由列表中填充/新增一条路由，并按照上述规则体进行表单操作：
  ![form](/imgs/v3/reference/admin/console-new/tag-rule/form.png)
4. 点击Yaml视图，可以看到表单视图中的操作修改同步到了yaml视图
  ![yaml](/imgs/v3/reference/admin/console-new/tag-rule/yaml.png)
5. 确认新增后，可以验证这个规则是否下发成功


## 删除标签路由规则
返回规则列表，在列表中找到要删除的指定规则，点击规则，删除即可：
![delte](/imgs/v3/reference/admin/console-new/tag-rule/delete.png)