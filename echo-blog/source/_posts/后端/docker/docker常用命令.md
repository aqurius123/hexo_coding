---
title: docker
categories:
- 后端
tags: docker
date: 2021-04-25
---

# docker常用命令
- docker images:常看所有镜像
- docker pull java:下载java镜像
- docker rmi 镜像名称：根据镜像名称删除指定镜像
- docker run 镜像名称：创建镜像并运行
- docker version：查看docker版本
- docker search java：查找带有java关键字的镜像仓库
- docker rm 容器名称：删除指定container
- dorcker rmi 镜像名称：删除指定镜像
- docker ps -a:查找对应运行过的container


## 删除指定镜像的步骤
1. 先使用docker ps -a查找到对应的container;
2. 再根据(docker stop 容器id)容器id停止container；
3. 再根据(docker rm 容器id)容器id删除对应的container;
4. 最后根据(docker rmi 镜像id)删除镜像；
