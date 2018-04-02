---
title: docker 学习浅谈
date: 2017-11-28 12:09:29
tags: 
	- docker
	- 部署
categories: 技术相关
---
# 简介

Docker 是一个包含多种工具的集合，它可以将你的项目或者服务进行打包分发，在任何地方运行你的代码而不依赖于系统环境。在系统和上层代码之间 docker container 为它们提供桥梁的作用，它提供本地文件系统，以及网络接口的访问。

由于最近有将整套代码服务进行打包分发的需求，我开始着手研究 docker 及其工具集。

# dockerize project

通过在项目根目录编写 Dockerfile 文件，将项目 “docker 化”。一般会通过 FROM 命令首先引入一个 base image, 然后在此基础上构建自己的 image。比如，我的项目使用 Python 写的，那么我可以使用 python:2.7 作为 base image。

在 Dockerfile 里面同样可以指定服务暴露的端口，即外部访问端口，以及环境变量等等。

<!--more-->

# docker compose 整合多种服务

在多数情况下，项目代码一般会依赖很多外部服务。比如，数据库，缓存服务，消息队列等等。docker 提供了 docker compose 工具将他们整合在一起。类似编写 Dockerfile 将项目 “docker 化”，通过编写 docker-compose.yml 文件进行服务整合。在 docker-compose.yml 中可以指定项目 app 是通过本地代码 build 还是来自 docker image。由于 docker 目前支持主流数据库及其他服务的 image，所以构建起来只需要指定相应 image，再配置好对应的文件系统路径即可，例如：

```
services:
   postgres:
     image: postgres:9.6
     ports:
      - '5432:5432'
     environment:
       - POSTGRES_USER=postgres
       - POSTGRES_PASSWORD=
       - POSTGRES_DB=magic
     volumes:
      - $PWD/docker/data/postgre_data
   redis:
     image: redis
     ports:
      - "6379:6379"
     volumes:
      - $PWD/docker/data/redis_data
```

完成 docker-compose.yml 后，只需要在当前目录运行 docker-compose up 即可将所以服务运行起来。

docker compose 还提供了其他命令便于开发者管理服务，例如 ps, logs 等等。

[docker compose 文档](https://docs.docker.com/compose/overview/)



