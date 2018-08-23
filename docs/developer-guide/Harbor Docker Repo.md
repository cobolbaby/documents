[TOC]

## Harbor搭建Docker镜像私服

### 目标

考虑到我们的目标是基于k8s构建容器集群管理平台，为了便于镜像的拉取，需要自建私服

### 背景

当前开发环境中已搭建了一套官方提供的私服服务，不过其仅提供镜像存储的服务，并未提供其他相关服务，比如：可视化管理，细力度的权限管控，镜像仓库间同步镜像，不能与K8S集成。为了解决其功能不足的问题，参考了业界比较成熟的实现方案，最终选择了vmware开源的Harbor搭建私有仓库服务

### 部署

#### 环境说明

- `CoreOS`
- `python`
- `docker-compose`

#### 安装脚本

```
#! /bin/bash
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.2.tgz
tar -zxvf harbor-offline-installer-v1.5.2.tgz -C /opt
# harbor.cfg 修改hostname、harbor_admin_password
cd /opt/harbor && ./install.sh
```

#### 服务启动

```
cd /opt/harbor
docker-compose up -d
```

#### 服务停止

```
cd /opt/harbor
docker-compose down
```

### 使用


#### 私服地址

私服地址: [http://harbor.inventec.com](http://harbor.inventec.com)

用户名: `admin`

密码: `123456`

> 备注: 当前阶段尚未接入`LADP`认证机制，如有建立个人账户的需求，可自行添加

#### 客户端配置

在拉取以及推送镜像前，客户端需要进行如下配置：

```
echo '{"insecure-registries": ["harbor.inventec.com"]}' > /etc/docker/daemon.json
```

#### 操作指南

- 拉取镜像

```
# 示例
docker pull harbor.inventec.com/development/python:2.7
```

- 上传镜像

```
# 示例
docker tag <IMAGEID> 
harbor.inventec.com/development/python:2.7
docker push harbor.inventec.com/development/python:2.7
```

> 备注: 在Push之前需要登录私服，以指定推送的地址 `docker login harbor.inventec.com`


- 权限管理

    配置指定用户对私有仓库的访问权限

- 同步镜像

    如果有多仓库间镜像同步的需求，可以使用该功能

### 填坑

`Harbor`安装依赖于`Python`执行环境，运行依赖于`docker-compose`指令，所以需要确保操作系统支持以上依赖。

而因为生产环境采用的操作系统为`CoreOS`，该系统与传统的`Linux`不同，没有内置`包管理器`/`Python`/`docker-compose`，所以如果要支持`Python`以及`docker-compose`也无法通过常规的包管理工具进行安装，只能另辟蹊径寻求他法了。

`CoreOS`作为容器`Linux`的代表，提倡所有的运行程序都打包在容器中。基于该核心理念，那我们可以采用`Python`
官方镜像执行外部的`Python`程序，不过需要进行一次封装，对外只提供`python`命令，而针对`docker-compose`，可以采用官方提供的 [容器安装方式](https://docs.docker.com/compose/install/#install-as-a-container)。


- python命令

```
#!/bin/sh
exec docker run -ti --rm -v "$PWD":"$PWD" -v /data:/data -w "$PWD" python:2.7-slim python "$@"
```
> 备注: 需要将封装的`python`指令放到`/opt/bin`目录下，因为`/opt/bin`是`CoreOS`给非系统内置指令提供的存储路径

### 致谢

感谢大家抽出宝贵的时间查看该文档