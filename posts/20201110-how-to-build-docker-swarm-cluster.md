# 记录从头搭建 Docker Swarm 集群的过程

> 发布日期：2020-11-10 21:56:16

即便 Swarm 不是容器编排的首选方案，但是了解它会使我们对服务编排工具有更深入的认识。

<!--more-->

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201110-how-to-build-docker-swarm-cluster/fuji-in-cloud.jpeg)

<center><font face="黑体" size=2>云中的富士山 | 2018-11-25 | 拍摄于 iphone 7p</font></center>

<br/>

一提到 Docker 的服务编排服务，大多数人首先想到的肯定是 Kubernetes，仿佛 Kubernetes 已经成为 Docker 编排的标准。确实， Kubernetes 可以帮我们做很多的事情，但同时它也非常重。如果我们只想要一个轻量级的 Docker 编排工具，并且容易搭建和使用，那么官方的 Docker Swarm 会是一个不错的选择。



## Docker Swarm 的特点

作为官方首推的 Docker 服务编排工具，其最终的特点就是“简单”。主要体现在下面几点：

- 对外以 Docker API 的方式呈现，因此学习成本很低；
- 轻量级，节约资源；
- 对 Docker 命令参数支持完善；



## 搭建 Swarm 集群

下面是 docker swarm 的架构图：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201110-how-to-build-docker-swarm-cluster/docker_swarm_arch.png)

#### STEP 01 创建服务器集群

准备三台安装好 Docker 和 docker-compose 的服务器(自己 aws 上的服务器)

- **Runner 3**:      172.31.47.251     |     52.37.216.32
- **Runner 2**:      172.31.46.208     |     34.223.240.14
- **All website**:   172.31.36.97       |     34.214.110.103

打开三台服务器上相应的端口

- **TCP port 2377** for cluster management communications
- **TCP** and **UDP port 7946** for communication among nodes
- **UDP port 4789** for overlay network traffic

#### STEP 02 运行部署命令

1. 在选定的 master 节点上运行 `sudo docker swarm init --advertise-addr 172.31.47.251` 来初始化 master 节点，然后我们会得到一个加入的token `SWMTKN-1-5sxf9hi55tnhush94ngq7q4j1b57l1qpazi8c5m8qxxel7hwdg-58dnnki56z18sgepaxh1rk0gs`

2. 然后在其他服务器上分别执行命令`docker swarm join --token SWMTKN-1-5sxf9hi55tnhush94ngq7q4j1b57l1qpazi8c5m8qxxel7hwdg-58dnnki56z18sgepaxh1rk0gs 52.37.216.32:2377` 让其他节点都加入到集群中来。

3. 这样我们就创建出了一个最基本的集群，我们可以用 `sudo docker node ls` 查看三个节点的状态

   ```
   ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
   f3efa8474eyyqzeoijr6huub3 *   ip-172-31-36-97     Ready               Active              Reachable           18.09.8
   g3k7dsddulw25uf9knu6c8vp6     ip-172-31-46-208    Ready               Active              Reachable           19.03.6
   zy8y7ye6m3mkg4pispooynj2v     ip-172-31-47-251    Ready               Active              Leader              19.03.11
   ```

#### STEP 03 配置集群的高可用

搭建完集群后，现在集群是一主多从的状态，如果主几点宕机后，整个集群就崩溃了，因此我们需要配置集群的高可用状态，配置方式也很简单，只需要在各个从节点上分别执行 `docker node promote NAME`  来提升各个节点的等级。这样如果主节点出问题后，那么 swarm 集群会通过选举的策略去选出其他的节点作为新的主节点。

#### docker swarm init 背后发生了什么

主要是 PKI 和安全相关的自动化

- 创建 swarm 集群的根证书
- manager 节点的证书
- 其它节点加入集群需要的 tokens

创建 Raft 数据库用于存储证书，配置，密码等数据



## 集群管理操作

#### 在集群上创建一个新的服务

`docker service create --name nginx --detach=false nginx`

`--detach=false` 表示是否在是在 background 模式执行

可以通过 `docker service inspect nginx` 去查看服务的具体信息

#### 查看服务

用 `docker service ls` 查看服务的运行

#### 查看日志

通过 `docker service logs -f S_NAME` 查看所有实例的日志

#### 修改服务配置

用 `docker service update --publish-add 8080:80 --detach=false nginx` 可以将 nginx 端口映射到宿主机的 8080 端口上

#### 服务伸缩

用 `docker service scale nginx=3 --detach=false` 来将 nginx 的服务变为三个，这样我们在进行访问的时候会通过 VIP LB 的方式做一个负载均衡，并且 LB 会带有一个 session 保存的功能

#### 停止服务

用 `docker service rm nginx` 可以停止掉一个服务



## Docker stack

正常的系统不肯能只有一个 service，往往是多个微服务的组合，服务和服务之间会有各种各样依赖关系，有些需要暴漏给用户访问，有些只是一些内部服务。而 Docker stack 在这里提供了一种配置文件的管理方式，使我们能更方便的管理我们的应用程序。

其实可以理解 stack 是一个 swarm 版本的 compose，一个用于生产和集群环境，一个用户测试和单机环境。

下面是一个简单的配置文件及其解释，使用的时候可以运行 `docker stack deploy -c service.yml NAME` 来部署服务，当配置文件有更新的时候，可以使用上面同样的命令进行服务的更新部署。

```yml
version: "3.4"                 # 版本号

services:                      # 定义一组服务
  alpine:                      # 服务名
    image: apline              # 所需镜像
    command:                   # 镜像启动的时候运行的命令
      - "ping"
      - "www.google.com"
    networks:                  # 所属网络
      - "test-overlay"
    deploy:                    # 部署配置
      endpoint_mode: dnsrr     # 访问模式
      replicas: 2              # 几个副本
      restart_policy:          # 重启策略
        condition: on-failure
      resources:               # 资源约束
        limits:
          cpus: "0.1"
          memory: 50M
    depends_on:                # 服务依赖
      - nginx

    nginx:
      image: nginx
      networks:
        - "test-overlay"
      ports:
        - "8080:80"

networks:                      # 定义一组网络
  test-overlay:                # 网络名字
    external: true             # 如果已经存在，则这样写
```



## 服务发现和网络

对于理解 swarm 的网络来讲，有两个最重要的点：

- 第一是外部如何访问部署运行在 swarm 集群内的服务，可以称之为入方向流量，在 swarm 里我们通过 ingress 来解决。
- 第二是部署在 swarm 集群里的服务，如何对外进行访问，这部分又分为两块：
  - 第一，东西向流量，也就是不同 swarm 节点上的容器之间如何通信，swarm 通过 overlay 网络来解决；
  - 第二，南北向流量 ，也就是 swarm 集群里的容器如何对外访问，比如互联网，这个是 `Linux bridge + iptables NAT` 来解决的；

#### 对外进行访问

通过一个例子来说明，见下图示例图：

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201110-how-to-build-docker-swarm-cluster/swarm_overlay_net_example.png)

每个容器都有两个网卡，eth1 和 eth0。

- 容器之间访问的时候，走的是 eth0 网卡，也就是链接到了 mynet 这个 overlay 网络上。
- 访问外网的时候，则是通过网卡 eth1，也就是链接到了宿主机的 bridge 网络上，然后通过 nat 技术对外进行通讯。

overlay 网络就是在物理网络之上的一个虚拟网络，使我们的应用不再依赖物理网络，又能保持物理网络不变。

#### 外网访问集群服务

docker swarm 的 ingress 网络叫 Ingress Routing Mesh，主要是为了实现把 service 的服务端口对外发布出去，让其能够被外部网络访问到。

ingress routing mesh是docker swarm网络里最复杂的一部分内容，包括多方面的内容：

- iptables 的 Destination NAT 流量转发
- Linux bridge，network namespace
- 使用 IPVS 技术做负载均衡
- 包括容器间的通信（overlay）和入方向流量的端口转发

首先我们先启动一个 service

```
docker service create --name web --network mynet -p 8080:80 --replicas 2 containous/whoami
# containous/whoami 可以帮助我们快速启动一个测试用的 web 服务
```

这样我们把启动的两个服务都映射到了本地的 8080 端口，我们可以在任何一台机器上访问 8080 端口，我们会发现每次访问都是在这两个 replicas 中进行轮转的，这就说明了中间有一层负载均衡。

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201110-how-to-build-docker-swarm-cluster/docker-swarm-ingress-network.png)

上图中，不管我们在哪台机器上进行访问，即便是最左侧的机器上没有 container，那么每台机器上的 VIP LB 也会将请求轮转到各个 container。

关于这里的负载均衡

- 这是一个 stateless load balancing
- 这是三层的负载均衡，不是四层的 LB is at OSI Layer 3(TCP)，not Layer 4 (DNS)
- 以上两个限制可以通过 Nginx 或者 HAProxy LB proxy 解决

#### 自定义网络

先创建一个 overlay 的网络 `docker network create --driver=overlay --attachable mynet`

将服务绑定到网络 `docker service create -p 80:80 --network=mynet --name nginx nginx`

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201110-how-to-build-docker-swarm-cluster/docker-swarm-custom-network.png)

#### 其他

- Swarm 的网络类型有 `vip` 和 `dnsrr`
- 网络的 Driver 的类型有 `bridge` `host` `overlay`，这和我们的虚拟机是对应的



## Docker secret 的使用

在程序部署的时候，有很多敏感数据，比如密码/证书之类的。Swarm 为我们提供了 secret 来保存敏感数据，主要有两种创建方式：

#### secret 的创建

第一种是通过标准输入创建

```
echo abc123 | docker secret create mysql_pass -
```

可以通过 `docker secret ls` 进行查看。

第二种是通过读取文件创建

```
docker secret create mysql_pass mysql_pass.txt
```

#### secret 的使用

我们可以通过下面命令进行使用

```
docker service create --name test --secret mysql_pass busybox ping 8.8.8.8
```

通过指定 secret 会在 container 中的 /run/secrets/ 目录下生成一个 secret 的文件，这样我们就可以在容器中进行使用了。

## Reference

[1] 从零开始，使用Docker Swarm部署集群教程: https://blog.csdn.net/u011936655/article/details/81147315  
[2] How to Add a Health Check to Your Docker Container: https://howchoo.com/devops/how-to-add-a-health-check-to-your-docker-container  
[3] Restarting an unhealthy docker container based on healthcheck: https://stackoverflow.com/questions/47088261/restarting-an-unhealthy-docker-container-based-on-healthcheck  
[4] set time period to restart a container with docker-compose: https://serverfault.com/questions/1003348/set-time-period-to-restart-a-container-with-docker-compose  
[5] Getting started with swarm mode: https://docs.docker.com/engine/swarm/swarm-tutorial/#open-protocols-and-ports-between-the-hosts  
[6] Docker Stack部署: https://blog.csdn.net/weixin_40364776/article/details/104432057

