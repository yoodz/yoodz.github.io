---
title: 基于Gitlab CI/CD + Docker的web前后端分离项目部署
tags:
  - 技术
  - docker
  - gitlab
  - ci/cd
description: 在部署一个web项目时，传统的部署方式是，首先在部署机安装所需要的软件，比如nginx，python，虚拟环境，mysql，redis,修改各种配置文件，然后把代码拷贝到部署机，调试启动项目。
abbrlink: bad899d7
date: 2020-04-26 22:53:36
---

## 背景介绍
在部署一个web项目时，传统的部署方式是，首先在部署机安装所需要的软件，比如nginx，python，虚拟环境，mysql，redis,修改各种配置文件，然后把代码拷贝到部署机，调试启动项目。如果要部署第二台机器，那就把上面的工作再做一遍。为了简化这样的部署流程，操作更流畅一点呢，我们采用了docker 容器话的部署方式，将用到的服务打包为镜像，然后上传到部署机，达成一键启动项目的爽快感。
这个流程最好是内网环境，内网部署Gitlab，内网启动Gitlab Runner，内网Docker私有库。内网的网速很快，提高部署效率。

## 前置条件
这些准备条件请自行Google。

- Gitlab 配置Gitlab Runner并开启CI/CD支持。
- Docker私有库搭建。
- nginx负载均衡策略
- docker安装、基本操作
- Dockerfile文件格式
- docker-comper容器编排


## 项目介绍
项目架构，前端是Vue的vue-admin-template后台框架，后端是Python的Flask框架，Python版本为3.6。部署方面，前端代码需要webpack打包编译，后端代码部署时要安装python 的依赖包。
总结：前端的代码用nginx部署，同时在请求后端接口时使用nginx做负载均衡。后端代码需要python3.6的环境，同时启动多个容器，来支持nginx的负载配置。


## 目录结构说明
![项目结构](https://static.afunny.top/2023/202304200853984.png)
- **gitlab-ci.yml** Gitlab的CI/CD的配置文件，配置了自动部署的各个阶段，后面会详细说明
- **docker-compose.yml **docker容器的编排工具，控制我们这里的两个镜像
- **docker文件夹 **里面是具体的dockerFile，前端、后端、nginx的配置
- **backend文件夹 **后端的python代码
- **其他文件 **其他的文件就是vue-admin-template 初始化的文件了



## 部署流程
![部署流程图](https://static.afunny.top/2023/202304200853034.png)
**流程解析**

1. 开发者将已经写好的代码提交到代码仓库
1. 代码仓库自动触发配置好的Runner机器，拉取代码。执行编译前端代码、打包镜像
1. Runner机器推送镜像到镜像仓库的操作，同时把Dockerfile推到部署机
1. 部署机根据DockerFile的配置从镜像仓库拉取最新的镜像，重启容器，完成部署


## 详细的构建部署流程

第一步： 在项目的根目录通过dockerfile构建镜像，镜像的名字命名规则为私有库的地址
![web-Dockerfile](https://static.afunny.top/2023/202304200853871.png)
![backend-Dockerfile](https://static.afunny.top/2023/202304200853891.png)
![nginx初始化配置](https://static.afunny.top/2023/202304200854081.png)

两个镜像的基础镜像都是拉了一份放在了内网，nginx的是为了加快下载速度，python是预先安装了一些依赖包。
主要的过程说明，web-Dockerfile里首先设定时区，避免出现时间相差八小时的情况，然后首先删除nginx原来的配置文件，然后复制代码文件，nginx配置文件到镜像内。backend-Dockerfile的配置和前一个大同小异。nginx 的配置文件里通过配置upstream来处理负载均衡，请求的分配原则为ip_hash，意思为根据客户端的ip，固定的去请求一个接口。具体的构建命令为：
```
docker build -f docker/web-Dockerfile -t harbor.gg.com/web/out_source_system_frontend .
docker build -f docker/backend-Dockerfile -t harbor.gg.com/web/out_source_system_backend .
```
注意最后的一个点！


第二步： 推送镜像到仓库
镜像打包完成之后，推送到私有仓库。
```
docker push harbor.gg.com/web/out_source_system_frontend
docker push harbor.gg.com/web/out_source_system_backen
```

第三步：部署机拉取镜像、启动容器
![docker-compose.yml](https://static.afunny.top/2023/202304200854246.png)

说明：这里的nginx 采用了主机模式，目的是为了访问宿主机的python接口。

- -f 指定Dockerfile文件路径
- -d 后台启动
```
docker-compose -f /var/www/html/out-source-system/docker-compose.yml up -d
```

## 其他
### Docker容器访问宿主机网络的方式
ngixn 容器配置负载均衡需要访问到宿主机的网络，这是后负载的配置有两种方式，

1. 在nginx 的配置表里直接配置宿主机的IP地址，
1. ngixn的容器网络模式选择主机模式，这样localhost就直接访问的是宿主机的ip



### 其他的命令
备选，docker build时指定网络模式
docker build --network=host -f docker/web-Dockerfile -t harbor.gg.com/web/out_source_system_frontend .
（可选）修改镜像的标签
docker tag out-source-system_2_backend:latest harbor.gg.com/web/out-source-system_2_backend:latest

## 完整的gitlab-ci.yml文件
```
stages: # 定义一批 stage
  - updateCode
  - frontendBuild
  - dockerBuild
  - dockerPush
  - deployToOrigin
updateCode: # 定义一个任务
  stage: updateCode # 相同 stage 的任务并发执行
  script: # runner 执行的叫脚本
    - sudo git-pull out-source-system git@gitlab.gg.com:platform/out-source-system.git
  only:
    - develop # 限制只有 master 分支的更新才会触发
  tags:
    - 10.10.243.134-tag # 对应 runner 配置的标签

frontendBuild:
  stage: frontendBuild
  script:
    - cd /home/gitlab-runner/out-source-system
    - sudo yarn
    - sudo yarn build:prod
  only:
    - develop
  tags: 
    - 10.10.243.134-tag

dockerBuild:
  stage: dockerBuild
  script:
    - cd /home/gitlab-runner/out-source-system
    - docker build -f docker/web-Dockerfile -t harbor.gg.com/web/out_source_system_frontend .
    - docker build -f docker/backend-Dockerfile -t harbor.gg.com/web/out_source_system_backend .
  only:
    - develop
  tags: 
    - 10.10.243.134-tag

dockerPush:
  stage: dockerPush
  script:
    - docker push harbor.gg.com/web/out_source_system_frontend
    - docker push harbor.gg.com/web/out_source_system_backend
  only:
    - develop
  tags: 
    - 10.10.243.134-tag

deployToOrigin:
  stage: deployToOrigin
  script:
    - sudo scp -r /home/gitlab-runner/out-source-system/docker-compose.yml root@10.10.243.154:/var/www/html/out-source-system/
    - sudo ssh root@10.10.243.154 "docker-compose -f /var/www/html/out-source-system/docker-compose.yml pull && docker-compose -f /var/www/html/out-source-system/docker-compose.yml up -d"
  only:
    - develop
  tags: 
    - 10.10.243.134-tag

```