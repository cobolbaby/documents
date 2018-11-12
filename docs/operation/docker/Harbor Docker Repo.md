[TOC]

## Harbor搭建Docker镜像私服

### 目标

- 避免手工导入导出镜像
- 可视化管理镜像资源
- 多数据中心镜像同步

### 选型

- **[Docker Registry](https://docs.docker.com/registry/)**

    * 提供基础的仓库服务

- **[Harbor](https://github.com/goharbor/harbor)**

    * 提供基础的仓库服务
    * 支持可视化管理镜像资源
    * 支持多仓库间镜像同步
    * 支持与K8S集成
    * 支持基于用户角色的权限管理
    * 使用广泛，文档较为完善

- **[Gitlab](https://docs.gitlab.com/ce/administration/container_registry.html)**

    * 提供基础的仓库服务
    * 支持可视化管理镜像资源
    * 支持Gitlab CI/CD，一站式管理镜像
    * 除官方文档外文档很少

### 部署

#### 环境说明

- `CoreOS`
- `python`
- `docker-compose`

#### 安装脚本

```bash
#! /bin/bash
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.2.tgz
tar -zxvf harbor-offline-installer-v1.5.2.tgz -C /opt
# harbor.cfg 修改hostname、harbor_admin_password
cd /opt/harbor && ./install.sh
```

#### 服务启动

```bash
cd /opt/harbor
docker-compose up -d
```

#### 服务停止

```bash
cd /opt/harbor
docker-compose down
```

### 使用


#### 私服地址

- 私服地址: `harbor.inventec.com`

- 用户名: `admin`

- 密码: `123456`

> 当前尚未接入`LADP`认证机制，如有建立个人账户的需求，可自行添加

#### 客户端配置

在拉取以及推送镜像前，客户端需要进行如下配置:

```
echo '{"insecure-registries": ["harbor.inventec.com"]}' > /etc/docker/daemon.json
systemctl restart docker.service
```

如果遇到因配置的`Docker`代理引发的问题(有些企业内部无法直接访问外网，需要配置代理才能拉取镜像)，请参考下方配置:

```
sudo mkdir -p /etc/systemd/system/docker.service.d/
sudo echo '[Service]
Environment="HTTP_PROXY=http://[proxy-addr]:[proxy-port]/" "HTTPS_PROXY=https://[proxy-addr]:[proxy-port]/"
 "NO_PROXY=localhost,127.0.0.1,harbor.inventec.com"' > /etc/systemd/system/docker.service.d/http-proxy.conf
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

#### 操作指南

- 拉取镜像

```
docker pull harbor.inventec.com/development/python:2.7
```

- 上传镜像

```
docker tag <CONTAINER_ID> harbor.inventec.com/development/python:2.7
docker push harbor.inventec.com/development/python:2.7
```

> 在Push之前需要登录私服，以指定推送的地址 `docker login harbor.inventec.com`

- 查看镜像

- 同步镜像

    如果有多仓库间镜像同步的需求，可以使用该功能

- 权限管理

    配置指定用户对私有仓库的访问权限

### 填坑

#### python命令

`Harbor`安装依赖于`Python`执行环境，运行依赖于`docker-compose`指令，所以需要确保操作系统支持以上依赖。

而因为生产环境采用的操作系统为`CoreOS`，该系统与传统的`Linux`不同，没有内置`包管理器`/`Python`/`docker-compose`，所以如果要支持`Python`以及`docker-compose`也无法通过常规的包管理工具进行安装，只能另辟蹊径寻求他法了。

`CoreOS`作为容器`Linux`的代表，提倡所有的运行程序都打包在容器中。基于该核心理念，那我们可以采用`Python`
官方镜像执行`Python`程序，如果需要支持`python`命令，则需要进行包装。

```bash
#!/bin/sh
exec docker run -ti --rm -v "$PWD":"$PWD" -v /data:/data -w "$PWD" python:2.7-slim python "$@"
```

#### docker-compose命令

针对`docker-compose`，可以采用官方提供的 [容器安装方式](https://docs.docker.com/compose/install/#install-as-a-container)

#### 生成证书失败

[install.sh Fail to generate key file](https://github.com/goharbor/harbor/issues/2920)

#### 测试连接成功但同步失败

[replication error，Test connection success](https://github.com/vmware/harbor/issues/3856)
