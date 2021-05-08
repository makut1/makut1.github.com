---
title: k8s operator基本概念
layout: post
guid: urn:uuid:2ef3550f-8cf3-400b-a55b-c512c9af8b2d
tags : [technology]
---

{% include JB/setup %}




1. operator出现的背景
2. 什么是有状态应用？
3. operator是什么？
4. 什么是crd（Custom Resource Definition） 和CR（Custom Resource）定制资源？
5. 什么是定制控制器？
6. operator的工作原理
7. etcd operator实践
8. 参考资料

### operator出现的背景

![img](/media/files/operator01.png)

### 什么是有状态应用？

分布式应用，它的多个实例之间，往往有依赖关系，比如：主从关系、主备关系。还有就是数据存储类应用，它的多个实例，往往都会在本地磁盘上保存一份数据。而这些实例一旦被杀掉，即便重建出来，实例与数据之间的对应关系也已经丢失，从而导致应用失败。所以，这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态应用” 。

例子：mysql“有状态应用”。

- 是一个“主从复制”（Maser-Slave Replication）的 MySQL 集群；
- 有 1 个主节点（Master）；
- 有多个从节点（Slave）；
- 从节点需要能水平扩展；
- 所有的写操作，只能在主节点上执行；
- 读操作可以在所有节点上执行。

### operator的定义：

operator是kubernetes的一个扩展，它使用自定义资源(Custom Resources)来管理应用和组件，并且遵循kubernetes的规范。它的核心就是自己编写控制器来实现自动化任务的效果，从而取代kubernetes自己的控制器和CRD资源。

简单来说operator就是特殊的controller，用来管理复杂的分布式应用

-  custom resource definition (CRD)
-  custom controller

使用operator是实现自动化的需求大概有以下几类：

- 按照需求部署一个应用
- 获取或者恢复一个应用的状态
- 应用代码升级，同时关联的数据库或者配置等一并升级
- 发布一个服务，让不支持kubernetes api的应用也能发现它
- 模拟集群的故障以测试集群稳定性
- 为分布式集群选取leader节点

### 什么是crd和cr？

资源（resource）是kubernetes api中的一个端点，其中存储的是某个类别的api对象的一个聚合。例如内置的pods资源包含一组pod对象。

定制资源（custom resource）是对kubernetes api的扩展，不一定在默认的kubernetes安装中就可用。定制资源所代表的是对特定kubernetes安装的定制。不过，很多kubernetes核心功能现在都用定制资源来实现，这使得kubernetes更加模块化。

定制资源可以通过动态注册的方式在运行中的集群内或出现或消失，集群管理员可以独立于集群 更新定制资源。一旦某定制资源被安装，用户可以使用 [kubectl](https://kubernetes.io/zh/docs/reference/kubectl/overview/) 来创建和访问其中的对象，就像他们为 *pods* 这种内置资源所做的一样。

![img](/media/files/operator03.png)

### 什么是定制控制器？

就定制资源本身而言，它只能用来存取结构化的数据。 当你将定制资源与 *定制控制器（Custom Controller）* 相结合时，定制资源就能够 提供真正的 *声明式 API（Declarative API）*。

使用[声明式 API](https://kubernetes.io/zh/docs/concepts/overview/kubernetes-api/)， 你可以 *声明* 或者设定你的资源的期望状态，并尝试让 Kubernetes 对象的当前状态 同步到其期望状态。控制器负责将结构化的数据解释为用户所期望状态的记录，并 持续地维护该状态。

你可以在一个运行中的集群上部署和更新定制控制器，这类操作与集群的生命周期无关。 定制控制器可以用于任何类别的资源，不过它们与定制资源结合起来时最为有效。 [Operator 模式](https://coreos.com/blog/introducing-operators.html)就是将定制资源 与定制控制器相结合的。你可以使用定制控制器来将特定于某应用的领域知识组织 起来，以编码的形式构造对 Kubernetes API 的扩展。

![img](/media/files/operator03.png)

### operator工作原理？

Operator 的工作原理，实际上是利用了 Kubernetes 的自定义 API 资源（CRD），来描述我们想要部署的“有状态应用”；然后在自定义控制器里，根据自定义 API 对象的变化，来完成具体的部署和运维工作。

### etcd operator实践：

1.create_role.sh 这个脚本是在为operator创建rbac访问权限 ，这里创建了ClusterRole(etcd-operator)和ClusterRoleBinding(etcd-operator),他们的namespace都是default。这样etcd-operator就具备了访问apiserver的权限。使用命令kubectl describe ClusterRole etcd-operator 即可查询其拥有的详情权限。

2.deployment.yaml  

其实就是一个deployment，replicas配置的是1，创建该对象也就是在创建etcd-operator。

创建operator时，应用会创建一个crd（custom Resource definition），这正是operator的特性。

查看crd使用命令kubectl get crd；查看crd的详细信息使用命令kubectl describe crd etcdclusters.etcd.database.coreos.com。

其作用：crd里面定义的group是etcd.database.coreos.com，kind是etcdcluster。有了这个crd，operator就可以作为一个控制器来对这个crd进行控制。

3.example-etcd-cluster.yaml 文件里定义的，正是 EtcdCluster 这个 CRD 的一个具体实例，也就是一个Custom Resource（CR）。该文件中的kind就是我们自定义的资源类型EtcdCluster,它其实就是crd的具体实现，即cr（custom resources）。

4.使用etcd operator的优势，就是可以把etcd静态组建集群的过程自动化。

etcd operator会根据size=3这个参数，动态组建一个etcd集群，首先启动一个节点，用上面的第一个命令，启动成功后，使用上面的第二个命令启动第二个节点，并且加入第一个节点的集群，重复这个过程只要pod数量达到了size的数量。从上面的initial-cluster字段我们能看出，operator组建的集群使用的是域名。

etcd operator创建集群的这个过程，其实就是operator对CRD的控制过程，operator控制的就是EtcdCluster类型的pod数量和EtcdCluster定义的size数量一致。

**deployment.yaml** 折叠源码

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: etcd-operator
spec:
  selector:
    matchLabels:
      app: etcd-operator
  replicas: 1
  template:
    metadata:
      labels:
        app: etcd-operator
    spec:
      containers:
      - name: etcd-operator
        image: docker-registry.xxx.virtual/xxx/etcd-operator:v0.9.4
        command:
        - etcd-operator
        - --create-crd=false
        env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
```

**example-etcd-cluster.yaml** 折叠源码

```
apiVersion: "etcd.database.coreos.com/v1beta2"
kind: "EtcdCluster"
metadata:
  name: "example-etcd-cluster"
  ## Adding this annotation make this cluster managed by clusterwide operators
  ## namespaced operators ignore it
  # annotations:
  #   etcd.database.coreos.com/scope: clusterwide
spec:
  size: 3
  version: "3.3.25" #这儿修改过，原始配置是3.2.13
```

### 参考资料

[kuberntes开发系列，运行第一个operator etcd](https://www.503error.com/2020/kubernete-开发系列-运行第一个operator-etcd/1781.html)

[扩展kubernetes](https://kubernetes.io/zh/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

[kubernetes编排技术八：使用operator管理有状态应用](https://blog.csdn.net/zjj2006/article/details/108338458)

[operator模式](https://kubernetes.io/zh/docs/concepts/extend-kubernetes/operator/)

[阅读coreos原文，其介绍了operator](https://coreos.com/blog/introducing-operators.html)

阅读这篇来自谷歌云的关于构建 Operator 最佳实践的 [文章](https://cloud.google.com/blog/products/containers-kubernetes/best-practices-for-building-kubernetes-operators-and-stateful-apps)