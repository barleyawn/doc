# Kubectl 命令使用

## kubectl 概述
> kubectl是Kubernetes集群的命令行工具，通过kubectl能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。运行kubectl命令的语法如下所示：

```
kubectl [command] [TYPE] [NAME] [flags]
```


## kubernetes 的资源类型

| 资源对象类型 |	缩略别名 |
|  ----  | ----  |
| apiservices	| | 
| certificatesigningrequests | csr |
| clusters	 |    |
| clusterrolebindings	 |  |
|   clusterroles	 |  |	 
| componentstatuses | 	cs |
| configmaps	| cm |
| controllerrevisions	| |  
| cronjobs	 |   |
| customresourcedefinition | 	crd |
| daemonsets |	ds |
| deployments   |	deploy |
| endpoints |	ep  |
| events |	ev |
| horizontalpodautoscalers  |	hpa |
|   ingresses |	ing |
| jobs	 | | 
| limitranges	| limits |
| namespaces    |	ns  |
| networkpolicies |	netpol |
| nodes	    | no    |
|   persistentvolumeclaims  |	pvc |
|   persistentvolumes	    |   pv  |
| poddisruptionbudget |	pdb |
| podpreset	 | |
|   pods    |	po  |
| podsecuritypolicies |	psp |
|   podtemplates	    |   | 
|   replicasets |	rs  |
|   replicationcontrollers  |	rc |
|   resourcequotas  |	quota   |
|   rolebindings	    |   | 
|   roles	    |   | 
|   secrets	 |  |
|   serviceaccounts |   sa  |
|   services	|   svc |
|   statefulsets	 |  |   
|   storageclasses	   |    |

### Apply 
通过文件名或控制台输入，对资源进行配置。

```
kubectl apply -f xxx.yaml
```


### Create 
命令通过文件或者stdin创建一个资源对象

```
kubectl Create -f xxx.yaml
```
### Get
获取指定或所有资源

```
kubectl get pods（指前面的资源）
kubectl get pod pod_name
```

### delete
用于删除集群中已存在的资源对象，可以通过指定名称、标签选择器、资源选择器等。
```
kubectl delete podname
```

### log
显示Pod中一个容器的日志。

```
kubectl logs podname
```

### exec
用于在Pod中的容器上执行一个命令， 与docker的exec 命令一样， 可以交互式的与pod中的容器进行通讯

```
# 执行nginx的容器中的  bash命令
kubectl exec -it nginx-c5cff9dcc-dr88w /bin/bash
```

### scale
为 Deployment, ReplicaSet, Replication Controller 或者 Job 设置一个新的副本数量

```
# 调整rs/foo的Pod的副本数量为3
kubectl scale --replicas=3 rs/foo 
```

### describe
用于显示一个或多个资源对象的详细信息，在这里通过获取上述nginx部署的信息。
```
kubectl describe deployments/nginx
```

## 资源描述文件格式
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  revisionHistoryLimit: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      imagePullSecrets:
        - name: dc-hspfd
      containers:
      # 应用的镜像
      - image: nginx
        name: nginx
        imagePullPolicy: IfNotPresent
        # 应用的内部端口
        ports:
        - containerPort: 80
          name: nginx80
        # 持久化挂接位置，在docker中      
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx-data
        - mountPath: /etc/nginx
          name: nginx-conf
      volumes:
      # 宿主机上的目录
      - name: nginx-data
        nfs:
          path: /nfs/nginx
          server: 192.168.1.10
      - name: nginx-conf
        nfs:
          path: /k8s-nfs/nginx/conf
          server: 192.168.1.10

```
资源文件 可以 为  .json 或 .yaml 或 .yml文件，其中配置含意：

* apiVersion  api当前版本号
* kind: 资源类型  可Namespace、Pod、 Deployment、Service、Ingress等
* replicas 副本数量
* selector 调度选择器， 可以根据lable选择Node部署
* containers  容器配置
* image docker镜像
* imagePullPolicy  镜像拉取策略   IfNotPresent:本地不存在时拉取
* ports 指定暴露端口号
* volumeMounts 卷挂载目录
* volumes:  宿主机目录

### Service清单示例

```
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

* selector 选择器， 可通过其下边的配置 选择对应的 Pod容器
* ports 端口暴露
* port.protocol:  指明服务暴露采用的协议  tcp、 udp、http等

### ingress 示例
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
spec:
  rules:
  # 配置七层域名
  - host: foo.bar.com
    http:
      paths:
      # 配置Context Path
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      # 配置Context Path
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80

```
* rules  路由规则
* host  指定七层域名地址，通过该域名可直接访问到Service资源接口
* path  指定访问接口的前缀
* backend 指定 Service资源