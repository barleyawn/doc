# Kubernetes kubectl 与 Docker 命令关系

在本文中，我们将介绍Kubernetes命令行与docker-cli之间的一些关系。kubectl工具，对于使用过docker-cli 用户来说，kubectl的设计感觉会很熟悉，但是也有一些明显的区别。本文的每个小结都讲解了一个docker子命令，并且列出与kubectl相对应命令。

## docker run
如果运行nginx 并且暴露？使用kubectl run。

* 使用docker：
```
$ docker run -d --restart=always -e DOMAIN=cluster --name nginx-app -p 80:80 nginx
a9ec34d9878748d2f33dc20cb25c714ff21da8d40558b45bfaec9955859075d0
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                         NAMES
a9ec34d98787        nginx               "nginx -g 'daemon of   2 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp, 443/tcp   nginx-app 
```
* 使用kubectl：
```
# start the pod running nginx
$ kubectl run --image=nginx nginx-app --port=80 --env="DOMAIN=cluster"
deployment "nginx-app" created
```

kubectl run 在Kubernetes集群> = v1.2上将创建名是 “nginx-app”的 Deployment。如果运行的是低的版本，则会创建一个 replication controllers，如果要了解低版本方式，请使用--generator=run/v1，将会创建 replication controllers。查看kubectl run更多详情。现在，我们可以使用上面创建的Deployment来暴露一个新的服务：
```
# expose a port through with a service
$ kubectl expose deployment nginx-app --port=80 --name=nginx-http
service "nginx-http" exposed
```
使用kubectl创建一个Deployment，他能保证任何情况下有N个运行nginx 的pods（其中N是默认定义声明的副本数，默认为1个）。它同时会创建一个Services，使用选择器匹配Deployment’s selector。有关详细信息，请参阅快速入门。

默认情况下镜像在后台运行，类似于docker run -d ...如果要在前台运行，请使用：
```
kubectl run [-i] [--tty] --attach <name> --image=<image>
```
要删除Deployment （及其pod），使用 kubectl delete deployment <name>

## docker ps
如何列出当前运行的内容？参考kubectl get。

* 使用docker：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                         NAMES
a9ec34d98787        nginx               "nginx -g 'daemon of   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp, 443/tcp   nginx-app
```
* 与kubectl：
```
$ kubectl get po
NAME              READY     STATUS    RESTARTS   AGE
nginx-app-5jyvm   1/1       Running   0          1h
```
## docker attach
如何连接已经运行在容器中的进程？参考kubectl附件

* 使用docker：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                         NAMES
a9ec34d98787        nginx               "nginx -g 'daemon of   8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp, 443/tcp   nginx-app
$ docker attach a9ec34d98787
```
...
* 使用kubectl：
```
$ kubectl get pods
NAME              READY     STATUS    RESTARTS   AGE
nginx-app-5jyvm   1/1       Running   0          10m
$ kubectl attach -it nginx-app-5jyvm
...
docker exec
```
如何在容器中执行命令？参考kubectl exec。

* 使用docker：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                         NAMES
a9ec34d98787        nginx               "nginx -g 'daemon of   8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp, 443/tcp   nginx-app
$ docker exec a9ec34d98787 cat /etc/hostname
a9ec34d98787
```
* 使用kubectl：
```
$ kubectl get po
NAME              READY     STATUS    RESTARTS   AGE
nginx-app-5jyvm   1/1       Running   0          10m
$ kubectl exec nginx-app-5jyvm -- cat /etc/hostname
nginx-app-5jyvm
```
交互式命令呢？

* 使用docker：
```
$ docker exec -ti a9ec34d98787 /bin/sh
# exit
```
* 使用kubectl：
```
$ kubectl exec -ti nginx-app-5jyvm -- /bin/sh      
# exit
```
有关更多信息，请参阅获取运行容器中的Shell。

## docker logs
如何查看进程打印stdout / stderr？参考kubectl logs。

* 使用docker：
```
$ docker logs -f a9e
192.168.9.1 - - [14/Jul/2015:01:04:02 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
192.168.9.1 - - [14/Jul/2015:01:04:03 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
```

* 使用kubectl：
```
$ kubectl logs -f nginx-app-zibvs
10.240.63.110 - - [14/Jul/2015:01:09:01 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.26.0" "-"
10.240.63.110 - - [14/Jul/2015:01:09:02 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.26.0" "-"
```

感觉是时候普及下pods和containers之间的微小差异了; 默认情况下，如果进程退出，pods是不会终止，相反，它会重新启动该进程。这与docker run 配置--restart=always 选项有一个主要区别。要查看以前在Kubernetes中运行的输出，请运行如下：
```
$ kubectl logs --previous nginx-app-zibvs
10.240.63.110 - - [14/Jul/2015:01:09:01 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.26.0" "-"
10.240.63.110 - - [14/Jul/2015:01:09:02 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.26.0" "-"
```
有关详细信息，请参阅日志和群集监视。

## docker stop 和 docker rm
如何停止和删除正在运行的进程？参考kubectl delete。

* 使用docker
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                         NAMES
a9ec34d98787        nginx               "nginx -g 'daemon of   22 hours ago        Up 22 hours         0.0.0.0:80->80/tcp, 443/tcp   nginx-app
$ docker stop a9ec34d98787
a9ec34d98787
$ docker rm a9ec34d98787
a9ec34d98787
```

* 使用kubectl：

```
$ kubectl get deployment nginx-app
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-app   1         1         1            1           2m
$ kubectl get po -l run=nginx-app
NAME                         READY     STATUS    RESTARTS   AGE
nginx-app-2883164633-aklf7   1/1       Running   0          2m
$ kubectl delete deployment nginx-app
deployment "nginx-app" deleted
$ kubectl get po -l run=nginx-app
# Return nothing
```

注意，不要直接删除pod，使用kubectl请删除拥有该pod的Deployment。如果直接删除pod，则Deployment将会重新创建该pod。

## docker login
与docker login相比，在kubectl中没有直接相识的命令。如果对于Kubernetes在私有registry中使用感兴趣，请参阅使用私有Registry。

## docker version
如何获客户端和服务器的版本号？参考kubectl version。

* 使用docker：
```
$ docker version
Client version: 1.7.0
Client API version: 1.19
Go version (client): go1.4.2
Git commit (client): 0baf609
OS/Arch (client): linux/amd64
Server version: 1.7.0
Server API version: 1.19
Go version (server): go1.4.2
Git commit (server): 0baf609
OS/Arch (server): linux/amd64
```

* 使用kubectl：
```
$ kubectl version
Client Version: version.Info{Major:"0", Minor:"20.1", GitVersion:"v0.20.1", GitCommit:"", GitTreeState:"not a git tree"}
Server Version: version.Info{Major:"0", Minor:"21+", GitVersion:"v0.21.1-411-g32699e873ae1ca-dirty", GitCommit:"32699e873ae1caa01812e41de7eab28df4358ee4", GitTreeState:"dirty"}
```
## docker info
如何获取有关配置和环境信息？参考kubectl cluster-info。


* 使用docker：
```
$ docker info
Containers: 40
Images: 168
Storage Driver: aufs
 Root Dir: /usr/local/google/docker/aufs
 Backing Filesystem: extfs
 Dirs: 248
 Dirperm1 Supported: false
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.13.0-53-generic
Operating System: Ubuntu 14.04.2 LTS
CPUs: 12
Total Memory: 31.32 GiB
Name: k8s-is-fun.mtv.corp.google.com
ID: ADUV:GCYR:B3VJ:HMPO:LNPQ:KD5S:YKFQ:76VN:IANZ:7TFV:ZBF4:BYJO
WARNING: No swap limit support
```

* 使用kubectl：

```
$ kubectl cluster-info
Kubernetes master is running at https://108.59.85.141
KubeDNS is running at https://108.59.85.141/api/v1/namespaces/kube-system/services/kube-dns/proxy
KubeUI is running at https://108.59.85.141/api/v1/namespaces/kube-system/services/kube-ui/proxy
Grafana is running at https://108.59.85.141/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
Heapster is running at https://108.59.85.141/api/v1/namespaces/kube-system/services/monitoring-heapster/proxy
InfluxDB is running at https://108.59.85.141/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy
```