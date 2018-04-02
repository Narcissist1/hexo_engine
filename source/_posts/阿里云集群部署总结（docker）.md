---
title: 阿里云集群部署总结（docker）
date: 2018-03-20 11:06:00
tags:
	- docker
	- 集群
categories: 技术相关
---

由于最近用户量不断增多，以及出了几次事故。稳定性的保证要求对后端结构进行优化和升级。由于之前调研过阿里云集群，底层采用 docker 部署服务，可以自动伸缩容器和节点（后面发现里面还是有坑的😓）。

<!--more-->

# 结构

后端API全部部署在集群之上，集群通过一个负载均衡和外部或者其他内网ECS通讯。通过指定相应的CPU和内存检测指标自动增减容器个数和节点（主机）个数。

同时由于阿里云提供的外部[负载均衡](https://www.aliyun.com/product/slb?spm=5176.8142029.388261.259.12db6d3eeqxQmL)不支持绑定到一个节点内的多个容器上，所以实际上当容器自动伸缩的时候，新产生的容器并不能投入使用，因为会和旧容器的端口产生冲突。

例如，有一个提供 web 服务的容器，对外暴露8000端口，那么当采用这种直接绑定外部负载均衡的方法时，该节点（主机）内部只能有一个这样的容器，这就使得容器自动扩容失去了意义。所以需要在节点内部再搭建一个负载均衡，并通过该负载均衡将流量分发到所有提供相同服务的容器上，而内部负载均衡和外部负载均衡绑定到一个特定端口上。
[文档](https://help.aliyun.com/document_detail/50309.html?spm=a2c4g.11186623.6.693.d08dJa)
阿里云提供的这个负载均衡镜像是基于[HAProxy](http://cbonte.github.io/haproxy-dconv/configuration-1.5.html?spm=a2c4g.11186623.2.13.cdfzPy)

结构如下：

{% qnimg Image6.png title:拓扑结构图 %}

# 模板

阿里云使用 docker 部署相应的服务到集群当中。同时阿里云还提供了一个叫模板的工具（定制化的 docker compose）来将你的一系列服务整合成一个应用进行管理。

模板管理相应的容器启动属性，以及网络设置，容器伸缩策略等等。
下面是一个 websocket 和 celery 应用的模板示例。

```
websocket:
  image: 'your_image_url'
  restart: always
  environment:
    - EXAMPLE_ENV=1
  ports:
    - '5000:5000/tcp'
  labels:
    aliyun.scale: '1'
    aliyun.global: 'true'
    aliyun.rolling_updates: 'true'
    aliyun.lb.port_5000: 'tcp://lb-2ze0e2wlvf9l1hfv9m1lv:5000'
  volumes:
    - /etc/localtime:/etc/localtime:ro
celery:
  image: 'your_image_url'
  command:
    - celery
    - '-A'
    - app
    - worker
    - '--loglevel=info'
  restart: always
  environment:
    - LANG=C.UTF-8
  cpu_shares: 100
  labels:
    aliyun.scale: '1'
    aliyun.rolling_updates: 'true'
    aliyun.auto_scaling.max_cpu: '70'
    aliyun.auto_scaling.min_cpu: '30'
    aliyun.auto_scaling.step: '1'
    aliyun.auto_scaling.max_instances: '10'
    aliyun.auto_scaling.min_instances: '2'
```

由于模板基于 docker compose, 所以很多 docker compose 的配置写法是可以直接使用的，不过有一些版本上的限制。

# 代码更新以及应用部署

## 镜像管理

阿里云提供了镜像托管服务，支持绑定各种代码托管网站，用户可以设置 webhook，当某一分支的代码更新时，自动构建镜像。当然也可以手动构建，然后本地推送。新建一个镜像仓库，阿里云会分配出镜像地址，和正常的 docker 的构建和 push 没有差别。

## 代码部署

对于更新应用代码，阿里云提供了两种方式，分别是蓝绿发布和标准发布。

### 蓝绿发布

蓝绿发布是一种零宕机的应用更新策略。进行蓝绿发布时，应用的旧版本服务与新版本服务会同时并存，同一个应用不同版本的服务之间共享路由，通过调节路由权重的方式，可以实现不同版本服务之间的流量切换。验证无误后，可以通过发布确认的方式将应用的旧版本的服务删除；如果验证不通过，则进行发布回滚，应用的新版本会进行删除。

### 标准发布

标准发布会在发布时删除旧版本并上线新版本，服务会有短暂中断。

# 坑

对于需要使用阿里云其他服务（Redis or MongoDB）的应用来说，自动伸缩节点其实完全无法使用。虽然将集群和这些数据库服务都放置在了同一个VPC（虚拟专有网络）之内，自动扩容出来的节点（在VPC之内）仍然需要手动更新数据库的安全组才能够进行访问。而且每一种数据库都有自己的访问安全控制，必须要一个一个手动修改。

# 总结

之前也有调研使用过 docker，这次结构升级将它应用在生产环境还是有不少挑战。总体来说，使用 docker 部署应用，让流程更加标准化，服务更加稳定。当有节点出现问题的时候，可以快速反应，将应用一键部署到其他节点。







