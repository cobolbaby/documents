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

#### 1) Volume Mount

最最最常见的问题，如果在`docker-compose.yml`文件中指定了外部挂载目录，但宿主机尚未创建该目录，容器就会无法启动。

详细报错信息可`docker stack ps --no-trunc <STACKNAME>`指令查看

#### 2) Overlay Network

如果重启 Manager Node 所在宿主机，则可能会出现集群中容器网络无法互通的问题，比如 Manager Node 上的容器无法`ping`通 Worker Node 上运行的容器。还有节点更换IP也会造成集群网络出问题(服务治理异常)。此时最有效的方法是**重建网络**

除了网络稳定性不好之外，关于原生Overlay性能问题有待考量

#### 3) Rolling Update

当服务滚动更新后，服务有时可能会无法访问。最常见的是 Nginx 反向代理的服务，原因也比较简单： Nginx 启动时会将反向代理的服务地址进行`DNS`解析并且缓存，如果此时更新 Nginx 反代的服务，会造成服务`IP`的动态分配，如果`IP`有所变化，那就意味着 Nginx 无法按照原来解析的地址进行请求转发，造成服务无法访问的问题。所以如果是滚动更新 Nginx 反代的服务，建议在nginx.conf配置`resolver`

#### 4) Routing Mesh

Swarm部署的时候会涉及到一个配置 [endpoint_mode](https://docs.docker.com/compose/compose-file/#deploy)，作用引官方的说法:

> Specify a service discovery method for external clients connecting to a swarm.

有两个选项`vip`或`dnsrr`，分别代表着`Virtual IP`以及`DNS Round Robin`策略，默认的策略为`vip`。

> By default, swarm services which publish ports do so using the routing mesh. When you connect to a published port on any swarm node (whether it is running a given service or not), you are redirected to a worker which is running that service, transparently. Effectively, Docker acts as a load balancer for your swarm services. Services using the routing mesh are running in virtual IP (VIP) mode. Even a service running on each node (by means of the --mode global flag) uses the routing mesh. When using the routing mesh, there is no guarantee about which Docker node services client requests.
>
> To bypass the routing mesh, you can start a service using DNS Round Robin (DNSRR) mode, by setting the --endpoint-mode flag to dnsrr. You must run your own load balancer in front of the service. A DNS query for the service name on the Docker host returns a list of IP addresses for the nodes running the service. Configure your load balancer to consume this list and balance the traffic across the nodes.

但服务容器化的时候，难免会遇到一些比较特殊的存在。
比如`greenplum`，程序`socket`绑定的`host`为容器的`IP`而非`0.0.0.0`，当路由策略仍使用`vip`的话，请求数据包路由至某节点之后，解包发现目的地址与实际监听的地址不符，请求失败。

#### 5) Ingress Network

Ingress 解决的问题是如何对外提供服务，不过结果可能未必是自己想要的，因为每个Node都需要暴露开放端口，增大的安全风险。

> The ingress network is a special overlay network that facilitates load balancing among a service’s nodes. When **any swarm node receives a request on a published port**, it hands that request off to a module called IPVS. IPVS keeps track of all the IP addresses participating in that service, selects one of them, and routes the request to it, over the ingress network.

具体实现原理，可瞅一下宿主机的`sudo iptables -L -t nat --line-numbers -v -n`

```
Chain DOCKER-INGRESS (2 references)
num   pkts bytes target     prot opt in     out     source               destination         
1      279 14508 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:48082 to:172.19.0.2:48082
2      204 10608 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:48081 to:172.19.0.2:48081
3      193 10036 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:28082 to:172.19.0.2:28082
4      135  7020 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:28081 to:172.19.0.2:28081
5      112  5824 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:58082 to:172.19.0.2:58082
6      148  7696 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:58081 to:172.19.0.2:58081
7      272 14133 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:38082 to:172.19.0.2:38082
8      187  9724 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:38081 to:172.19.0.2:38081
9      174  9048 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:18082 to:172.19.0.2:18082
10     114  5928 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:18081 to:172.19.0.2:18081
11    2644  138K DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8081 to:172.19.0.2:8081
12       0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:7077 to:172.19.0.2:7077
13       0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:6066 to:172.19.0.2:6066
14    2533  132K DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:4040 to:172.19.0.2:4040
15       0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.19.0.2:8080
16    378K   22M DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:5432 to:172.19.0.2:5432
17     29M 3169M RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

可不可以配置一个外部负载均衡器，代理后端所有服务节点???

#### 6) Bind Persistent Volume

举个数据库的例子，数据库的多个容器实例分别绑定不同的存储数据。对于这些容器实例来说，第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间容器实例被重新创建过。因此往往需要将容器实例与持久化数据卷进行绑定。

#### 7) Cronjob

Docker官方并没有为定时任务提供很好的管控方式，所以很多时候还得借助于第三方。而如果仅依赖系统的crontab，管理起来又略显混乱。

#### 8) Software Ecosystem

很多优秀的软件并没有提供Swarm部署模式的官方支持。比如：Spark/Greenplum/Kong

#### 9) Microservices Sidecar Pattern

没有容器组的概念，所以Sidecar部署比较难落地。

#### 10) Failover

某节点中容器运行一段时间之后就重启，初步考虑有两个原因：

- 容器中程序异常退出

- 节点内存资源不足引发的OOM异常

    采用增加内存的方式验证

- 节点网络联通问题引发的故障切换

    先用`tcpdump`抓下数据包，如果`tcp`连接有问题，则用`mtr`测一下网络传输会不会有丢包，如果丢包率较高，换张网卡或者重插一下网线看看能否解决问题
    
    不过有篇文章说了另外一个观点：[不要开启tcp_tw_recycle](https://ieevee.com/tech/2017/07/19/tcp-tw-recycle.html)，也不知道说的对不对？

    
```
CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS                        PORTS               NAMES
813bfc1d4202        harbor.inventec.com/development/gpdb:4.3.25   "./entrypoint.sh"        4 hours ago         Exited (137) 3 hours ago                          gpdb_segment_sdw1.1.6mt9svwae6jhcadj3a5pmof7d
c7f5a6614f18        harbor.inventec.com/development/gpdb:4.3.25   "./entrypoint.sh"        4 hours ago         Exited (137) 4 hours ago                          gpdb_segment_sdw1.1.25vt3xzhrhkbxxlgkl4kp0781
```

#### 11) Keepalived

不支持，Keepalived并不能用Swarm部署，只能采用Host网络，而K8S官方就有支持。从中也可以看出K8S可以覆盖更多高可用的使用场景。

### 参考资料

- [官方文档](https://docs.docker.com/engine/swarm/)
- [Docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/swarm_mode/)
- [CloudMan](https://www.cnblogs.com/CloudMan6/tag/Swarm/)
- [Overlay Network Driver on Windows ](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)
- [Swarm Mode: Overlay networks intermittently stop working](https://forums.docker.com/t/docker-worker-nodes-shown-as-down-after-re-start/22329) 宿主机重启的危害
- [Nginx does not automatically pick up DNS changes in Swarm](https://stackoverflow.com/questions/46660436/nginx-does-not-automatically-pick-up-dns-changes-in-swarm)
- [Configure Service Discovery](https://docs.docker.com/v17.09/engine/swarm/networking/#configure-service-discovery)
- [Swarm Native Service Discovery](https://success.docker.com/article/networking)
- [探索 Docker Bridge 的正确姿势](http://blog.daocloud.io/docker-bridge/) 了解一下DNAT/SNAT
- [Kubernetes NodePort vs LoadBalancer vs Ingress](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)
- [Think about NodePort in Kubernetes](https://oteemo.com/2017/12/12/think-nodeport-kubernetes/)
- [Swarm mode should support batch/cron jobs](https://github.com/moby/moby/issues/23880)
- [Benchmark results of Kubernetes network plugins](https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-36475925a560)
- [Docker swarm mode - connection refused when binding to overlay network interface](https://github.com/moby/moby/issues/30874) endpoint_mode配置
- [Binding Persistent Volumes by Labels](https://docs.okd.io/latest/install_config/storage_examples/binding_pv_by_label.html#binding-pv-by-label-pv-with-labels)
- [开启iptables情况下Swarm、Kubernetes等组件正常工作的配置](https://blog.csdn.net/A632189007/article/details/78909835)
- [Docker Swarm添加了太多iptables规则，且很多重复的，怎么处理](https://segmentfault.com/q/1010000015272794?_ea=3861870)
- [我的容器崩掉了](https://wumingxiaozu.com/2016/01/22/ComputerScience/Docker/%E6%88%91%E7%9A%84Docker%E7%A8%8B%E5%BA%8F%E5%B4%A9%E6%8E%89%E4%BA%86/)
- [Out Of Memory Exceptions (OOME)](https://docs.docker.com/config/daemon/#out-of-memory-exceptions-oome)
- [Docker Swarm管理节点高可用分析](http://zhoujinl.github.io/2018/10/19/docker-swarm-manager-ha/)
- [Bypass the routing mesh for a swarm service](https://docs.docker.com/network/overlay/) 两种路由策略
- [Blocking ingress traffic to Docker swarm worker machines](https://ops.tips/blog/blocking-ingress-traffic-to-docker-swarm-worker-machines/) Ingress网络安全
- [How to run docker-compose on remote host](https://stackoverflow.com/questions/35433147/how-to-run-docker-compose-on-remote-host) 使用docker-compose将容器指定部署到远程宿主机
