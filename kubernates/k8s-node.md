# Kubernetes Nodes 
Node是Kubernetes中的工作节点，最开始被称为minion。一个Node可以是VM或物理机。每个Node（节点）具有运行pod的一些必要服务，并由Master组件进行管理，Node节点上的服务包括Docker、kubelet和kube-proxy。有关更多详细信息，请参考架构设计文档中的“Kubernetes Node”部分。

## Node Status
节点的状态信息包含：

* Addresses
* Phase (已弃用)
* Condition
* Capacity
* Info


## Addresses
这些字段的使用取决于云提供商或裸机配置。

* HostName：可以通过kubelet 中 --hostname-override参数覆盖。
* ExternalIP：可以被集群外部路由到的IP。
* InternalIP：只能在集群内进行路由的节点的IP地址。

## Condition
conditions字段描述所有Running节点的状态。
```
"conditions": [
  {
    "kind": "Ready",
    "status": "True"
  }
]
```

如果Ready condition的Status是“Unknown” 或 “False”，比“pod-eviction-timeout”的时间长，则传递给“ kube-controller-manager”的参数，该节点上的所有Pod都将被节点控制器删除。默认的eviction timeout时间为5分钟。在某些情况下，当节点无法访问时，apiserver将无法与kubelet通信，删除Pod的需求不会传递到kubelet，直到重新与apiserver建立通信，这种情况下，计划删除的Pod会继续在划分的节点上运行。

在Kubernetes 1.5之前的版本中，节点控制器将强制从apiserver中删除这些不可达（上述情况）的pod。但是，在1.5及更高版本中，节点控制器在确认它们已经停止在集群中运行之前，不会强制删除Pod。可以看到这些可能在不可达节点上运行的pod处于"Terminating"或 “Unknown”。如果节点永久退出集群，Kubernetes是无法从底层基础架构辨别出来，则集群管理员需要手动删除节点对象，从Kubernetes删除节点对象会导致运行在上面的所有Pod对象从apiserver中删除，最终将会释放names。


## Capacity

描述节点上可用的资源：CPU、内存和可以调度到节点上的最大pod数。

## Info

关于节点的一些基础信息，如内核版本、Kubernetes版本（kubelet和kube-proxy版本）、Docker版本（如果有使用）、OS名称等。信息由Kubelet从节点收集。
