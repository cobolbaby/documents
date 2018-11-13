[TOC]

## Swarm目录

### docker-compose是什么

docker-compose 是用于容器编排的工具，通过一个配置文件定义多容器的应用，然后使用 docker-compose 脚本来启动，停止和重启应用，替代了自行编写 shell 脚本，使部署更加标准化。

### docker-compose的不足

但 docker-compose 有一个很大的问题就是只能部署单节点，如果要部署在生产环境中，面向集群多节点，就力不从心了，此时就会涉及到一个概念 ---- **容器集群管理**

### 容器集群管理

容器集群管理必须具有集群管理以及容器编排的功能，而当下有三种普遍被接受的方案:

- `Kubernetes(k8s)`
- `Docker Swarm`
- `Apache Mesos`

`Swarm` 擅长的是跟 `Docker` 生态的无缝集成，将容器编排和集群管理功能全部内置到 `Docker` 项目，用户只需简单的几条命令就能创建集群加入集群，在集群上部署。

而相比于 `Swarm`, `k8s` 就显得复杂太多了，不过却是未来的目标

### Swarm基础概念

![swarm-architecture](https://msdnshared.blob.core.windows.net/media/2017/02/SwarmOverlayFunctionalView-1024x811.png)

#### Swarm Mode

从 Docker v1.12 开始，集群管理和编排功能已经集成进 Docker Engine。当 Docker Engine 执行 Swarm 初始化或者加入到一个存在的 Swarm 集群中时，它就启动了`Swarm Mode` 。

没启用 Swarm Mode 时，Docker 只能执行常规容器命令; 启用之后，Docker 增加了编排 `Service` 的能力。

#### Node

Swarm 集群中一个 Docker Engine 就是一个 Node，有两种类型: `Manager` 和 `Worker`。

部署应用的时候，我们需要在 Manager Node 上执行部署命令，Manager Node 会将部署任务拆解并分配给一个或多个 Worker Node 完成部署。

Manager Node 负责执行编排和集群管理工作。Swarm 中如果有多个 Manager Node ，它们会自动协商并选举出一个Leader 执行编排任务。

Worker Node 接受并执行派发的任务。默认配置下 Manager Node 同时也是一个 Worker Node ，不过可以将其配置成 `Manager-Only Node`，让其专职负责编排和集群管理工作。

![swarm-node](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)

#### Service

Service 定义了 Worker Node 上要执行的任务。

举一个 Service 的例子：在 Swarm 中启动一个 Nginx 服务，使用的镜像是 nginx:latest，为支持高可用，副本数为 3。

Manager Node 负责创建这个 Service，经过分析知道需要启动 3 个 nginx 容器，根据当前各 Worker Node 的状态将运行容器的任务分配下去，比如 Worker1 上运行两个容器，Worker2上运行一个容器。

运行了一段时间，Worker2 突然宕机了，Manager Node 监控到这个故障，于是立即在 Worker3 上启动了一个新的 nginx 容器。这样就保证了 Service 处于期望的三个副本状态。

![swarm-service](https://docs.docker.com/engine/swarm/images/services-diagram.png)

#### Stack

Stack 定义了由若干 Service 构成的服务堆栈，用于描述一个完整的应用。

### Swarm常用指令

#### 初始化

```bash
# Manager Node 执行
docker swarm init --advertise-addr 10.190.5.110
# Worker Node 执行
docker swarm join --token SWMTKN-1-44xebmqerko0v8y3mxlaz00xc6supwol8ub4sbs9kvtl1k2rv3-1o9ohpmzpmapl5atgi069w017 10.190.5.110:2377
```

#### 查看节点

```bash
# Manager Node 执行
docker node ls
```

#### 初始化网络

```bash
docker network create --driver overlay --subnet=10.11.0.0/16 --attachable <NETWORKNAME>
```

#### 编写compose.yml文件

示例如下:

```
version: "3.3"
services:
  mdw:
    image: ${REGISTRY}/${TAGNAME}
    hostname: mdw
    ports:
      - "5432:5432"
    volumes:
      - /opt/greenplum/config:/opt/greenplum/config
      - /disk1:/disk1
    deploy:
      mode: replicated
      replicas: 1
      # resources:
      #   limits:
      #     cpus: "0.1"
      #     memory: 50M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints: 
          - node.role == manager

networks:
  default:
    external:
      name: gpdb
```

#### 启动服务

```bash
docker stack deploy -c docker-compose.yml <STACKNAME>
```

#### 查看状态

```bash
docker stack ps --no-trunc <STACKNAME>
docker service ls
```

#### 扩容缩容

```bash
docker service scale <SERVICENAME>=5
```

#### 关闭服务

```bash
docker stack rm <STACKNAME>
```

####  更新服务

```bash
docker service update <STACKNAME_SERVICENAME> --image harbor.inventec.com/development/nginx:latest
```

### Swarm集群部署

集群部署难免重复的工作，秉持着重复的工作让计算机去做的原则，在此引入自动化运维工具`Ansible`，简化搭建`Swarm`集群过程中的手工操作，同时该工具也能在日常的集群管理工作中起到极大的辅助作用。

### Swarm常见问题

#### 挂载目录

最最最常见的问题，如果在`docker-compose.yml`文件中指定了外部挂载目录，但宿主机尚未创建该目录，容器就会无法启动。

详细报错信息可`docker stack ps --no-trunc <STACKNAME>`指令查看

#### 网络问题

如果重启 Manager Node 所在宿主机，则可能会出现集群中容器网络无法互通的问题，比如 Manager Node 上的容器无法`ping`通 Worker Node 上运行的容器，此时最有效的方法是**重建网络**

#### 服务更新

当服务滚动更新后，服务有时可能会无法访问。最常见的是 Nginx 反向代理的服务，原因也比较简单： Nginx 启动时会将反向代理的服务地址进行`DNS`解析并且缓存，如果此时更新 Nginx 反代的服务，会造成服务`IP`的动态分配，如果`IP`有所变化，那就意味着 Nginx 无法按照原来解析的地址进行请求转发，造成服务无法访问的问题。所以如果是滚动更新 Nginx 反代的服务，建议同时对 Nginx 热重载

#### 负载策略

Swarm 服务有一个`endpoint_mode`配置来设置负载均衡的策略，可以选择`vip`或`dnsrr`，分别代表着`Virtual IP`以及`DNS Round Robin`负载均衡策略，默认的策略为`vip`。但服务容器化的时候，难免会遇到一些比较特殊的存在，比如`greenplum`，因其`socket`绑定的`host`为容器的`IP`而非`0.0.0.0`，所以此时如果仍然使用`vip`的负载策略，就会造成`greenplum`数据节点无法提供服务。

#### 固定节点

对于有状态的容器，比如数据库，在做集群部署的时候为了确保持久化的数据被加载，此时就需要保证在 Worker Node 上有固定的容器在运行。为此就需要在`docker-compose.yml`配置文件中明确指定容器固定部署于哪个节点上。

#### 跨节点数据卷

`cross-host mounted volumes`被指很不可靠，所以从一开始就不建议使用该特性

### 参考资料

- [官方文档](https://docs.docker.com/engine/swarm/)
- [Docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/swarm_mode/)
- [CloudMan](https://www.cnblogs.com/CloudMan6/tag/Swarm/)
- [Overlay Network Driver on Windows ](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)