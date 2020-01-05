# ** kubernetes 初识 **
## 一、kubernetes的介绍
>Kubernetes是Google开源的一个容器编排引擎，它支持自动化部署、大规模可伸缩、应用容器化管理。在生产环境中部署一个应用程序时，通常要部署该应用的多个实例以便对应用请求进行负载均衡。

### kubernetes能做什么
> 1、自动化容器的部署和复制

> 2、随时扩展或收缩容器规模

> 3、将容器组织成组，并且提供容器间的负载均衡

> 4、很容易地升级应用程序容器的新版本

> 5、弹性扩容

## 集群
> 集群是一组节点，这些节点可以是物理服务器或者虚拟机，之上安装了Kubernetes平台。下图展示这样的集群。注意该图为了强调核心概念有所简化。这里可以看到一个典型的Kubernetes架构图

![架构](http://dockone.io/uploads/article/20190625/d7ce07842371eab180725bab5164ec17.png)

## Pod
> pod, 豌豆夹，每一个豌豆荚有多个槽，每个槽都一颗豌豆。顾名思义，我们把一个个docker容器当做一颗豌豆，然后将所有豌豆(容器)打包成一个组，安排在节点上。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。

## Service
>   对于Pod而言，它是一组容器的集合，而容器又是短暂的，那么每个容器的IP就会是变化的，那我们又如何能正确的访问到这些容器呢？  

>   Service是定义一系列Pod以及访问这些Pod的策略的一层抽象。Service通过Label找到Pod组。那Service是如何确保我们能正确访问容器呢？

    创建Service时会指定名称, 这时Service会创建一个本地集群的DNS入口，我们只需要通过DNS查找主机名为 ‘test’，就能够解析出前端应用程序可用的IP地址。现在前端已经得到了后台服务的IP地址，但是它应该访问2个后台Pod的哪一个呢？Service在这2个后台Pod之间提供透明的负载均衡，会将请求分发给其中的任意一个。通过每个Node上运行的代理（kube-proxy）完成。

![service](http://dockone.io/uploads/article/20190625/e7a273fcdc03d2417b354b60c253552f.gif)


## Master 
> 集群拥有一个Kubernetes Master, 供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Replication Controller。

## Node
> node :节点， 指的是一个物理或虚拟服务器，作为kubernetes中的Kubernetes worker。每个节点，包含：

* kubelet: 表示主节点代理, 与Master节点进行交互，并将该Node交由Master API Service 进行管理。
* Kube-proxy： Service使用其将链接路由到Pod
* kubectl: 可通过该命令去管理 Pod、Service的创建与销毁
