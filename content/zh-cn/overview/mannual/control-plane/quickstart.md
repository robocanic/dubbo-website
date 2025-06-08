---
aliases:
  - /zh/overview/reference/admin/quickstart/
  - /zh-cn/overview/reference/admin/quickstart/
description: ""
linkTitle: 快速开始
no_list: true
title: 安装Admin并开始使用
type: docs
weight: 1
working_in_progress: true
---

## 部署Dubbo Admin

> 本安装指南使用命令行工具 dubboctl
### 先决条件
开始之前，检查下列先决条件
1. 下载 Dubboctl 发行版
2. 准备一个k8s集群

### 使用dubboctl安装 Dubbo Admin
使用默认配置安装 Dubbo Admin 
```sh
dubboctl install -y
```
此命令将以默认配置在 Kubernetes 集群上安装dubbo admin。你也可以选取任意一个 dubbo admin内置的`profile`
```sh
dubboctl install --set profile=demo -y
```
可以选取任意一个 dubbo admin内置的参数
```
dubboctl install --set values.nacos.enabled=true
```
> 在 dubboctl 中使用 --set 参数跟使用 Helm 一样且当前 Helm 的 values.yaml API 向后兼容。唯一的区别是必须给原有 values.yaml 路径前面加上 values. 前缀，这是 Helm 透传 API 的前缀。
dubboctl 命令通过命令行选项支持完整 dubboOperator API
### 生成安装过程用到的清单文件
``` sh
dubboctl manifest generate > $HOME/generated-manifest.yaml
```
### 卸载
要从集群中完整卸载 Dubbo admin，运行下面命令
``` sh
dubboctl uninstall --remove -y
```
这条命令将移除所有 Dubbo 资源,后续版本将会支持指定文件。
命名空间 dubbo-system 默认不会被移除。如果不再需要可以用下面命令移除该命名空间
```sh
kubectl delete namespace dubbo-system
```
## 部署示例微服务集群
使用教程将围绕一个简单的线上商城微服务系统展开，演示资源管理和流量管控能力。

系统由 5 个微服务应用组成：
- Frontend 商城主页，作为与用户交互的 web 界面，通过调用 User、Detail、Order 等提供用户登录、商品展示和订单管理等服务。
- User 用户服务，负责用户数据管理、身份校验等。
- Order 订单服务，提供订订单创建、订单查询等服务，依赖 Detail 服务校验商品库存等信息。
- Detail 商品详情服务，展示商品详情信息，调用 Comment 服务展示用户对商品的评论记录。
- Comment 评论服务，管理用户对商品的评论数据。

### 部署商城系统
为方便起见，我们将整个系统部署在 Kubernetes 集群，执行以下命令即可完成商城项目部署，项目源码示例在 dubbo-samples/task。

``` sh
kubectl apply -f https://raw.githubusercontent.com/apache/dubbo-samples/master/10-task/dubbo-samples-shop/deploy/All.yml
```

执行以下命令，确定所有服务、Pod都已正常运行：
``` sh
$ kubectl get services -n dubbo-demo
$ kubectl get pods -n dubbo-demo
``` 

### 获取访问地址
执行以下命令，将集群端口映射到本地端口：
``` sh
kubectl port-forward -n dubbo-demo deployment/shop-frontend 8080:8080
```

```sh
kubectl port-forward -n dubbo-system service/dubbo-admin 8888:8888
```
此时，打开浏览器，即可通过以下地址访问：

● 商城首页 http://localhost:8080
● Dubbo Admin 控制台 http://localhost:8888/admin