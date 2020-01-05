# **[使用kubeadm安装kubernates 1.12](https://www.kubernetes.org.cn/3536.html)**
## 一、整体架构
![架构](https://d33wubrfki0l68.cloudfront.net/817bfdd83a524fed7342e77a26df18c87266b8f4/3da7c/images/docs/components-of-kubernetes.png)
> 如图，kubenetes包含以下组件：

### ETCD
ETCD，是一个受到ZooKeeper与doozer启发而催生的项目，是一个开源的、分布式的键值对数据存储系统，由Go语言实现，用于存储key-value键值对，同时不仅仅是存储，主要用途是提供共享配置及服务发现，使用Raft一致性算法来管理高度可用的复制日志。目前google已经在kubernetes全面使用etcd做为服务注册与发现服务。
* 简单：定义明确，面向用户的API（gRPC）
* 安全：具有可选客户端证书身份验证的自动TLS
* 快速：基准测试10,000次/秒
* 可靠：使用Raft正确分布

### Kubernetes API Server
API Server主要用来处理REST的操作，确保它们生效，并执行相关业务逻辑，以及更新etcd（或者其他存储）中的相关对象。API Server是所有REST命令的入口，它的相关结果状态将被保存在etcd（或其他存储）中。API Server的基本功能包括：

* REST语义，监控，持久化和一致性保证，API 版本控制，放弃和生效
* 内置准入控制语义，同步准入控制钩子，以及异步资源初始化
* API注册和发现
* 集群的网关

### Kube Controller Manager
用于执行大部分的集群层次的功能，它既执行生命周期功能(例如：命名空间创建和生命周期、事件垃圾收集、已终止垃圾收集、级联删除垃圾收集、node垃圾收集)，也执行API业务逻辑（例如：pod的弹性扩容）。控制管理提供自愈能力、扩容、应用生命周期管理、服务发现、路由、服务绑定和提供。Kubernetes默认提供Replication Controller、Node Controller、Namespace Controller、Service Controller、Endpoints Controller、Persistent Controller、DaemonSet Controller等控制器。

### Scheduler
scheduler组件为容器自动选择运行的主机。依据请求资源的可用性，服务请求的质量等约束条件，scheduler监控未绑定的pod，并将其绑定至特定的node节点。Kubernetes也支持用户自己提供的调度器，Scheduler负责根据调度策略自动将Pod部署到合适Node中，调度策略分为预选策略和优选策略。
* 预选Node：遍历集群中所有的Node，按照具体的预选策略筛选出符合要求的Node列表。如没有Node符合预选策略规则，该Pod就会被挂起，直到集群中出现符合要求的Node。

* 优选Node：预选Node列表的基础上，按照优选策略为待选的Node进行打分和排序，从中获取最优Node。

### 其它组件
* ingress 提供基于Http协议的路由转发机制，是一个nginx容器
* Fannel 或 kube-router 或 weave 基于overlay(虚拟网络 VXLAN)的网络组件
* kubernests-dashboard  可视化的web控制管理平台
* heapster + influxdb + grafana 容器集群监控和性能分析工具
* dnsmasq DNS服务

## 二、安装准备
k8s集群是由Master + Node节点组成。Master作为整体集群的控制中心，因此对于Master节点的高可用性和稳定性尤其重要。因为下面我们将搭建一个多Master的k8s服务集群。
### 1、系统准备
#### 1.1 如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看[Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)中的”Check required ports”一节。 这里简单起见在各节点禁用防火墙：
```
systemctl stop firewalld
systemctl disable firewalld
```
#### 1.2 设置内核，禁用SELINUX：
```
setenforce 0

sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config

cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
EOF
modprobe br_netfilter
sysctl -p /etc/sysctl.conf
```
#### 1.3 关闭swap
```
swapoff -a 
sed -i 's/.*swap.*/#&/' /etc/fstab
```
#### 1.4 安装常用软件
```
yum install -y wget net-tools \
            conntrack-tools \
            ntpdate
mkdir -p /opt/crontab 
echo '*/30 * * * * /usr/sbin/ntpdate time7.aliyun.com >/dev/null 2>&1' > /opt/crontab/ntpdate.crtab
crontab /opt/crontab/ntpdate.crtab
systemctl enable ntpdate.service && systemctl start ntpdate.service
```
1. [conntrack-tools](https://www.cnblogs.com/silvermagic/p/7666093.html):是一套Linux用户空间连接跟踪工具,用于系统管理员进行交互连接跟踪系统,该模块提供了iptables的状态数据包检查。它包括了用户空间的守护进程conntrackd和命令行界面conntrack。
2. [ntpdate](https://www.cnblogs.com/xxlu/archive/2017/06/09/6965548.html):当Linux服务器的时间不对的时候，可以使用ntpdate工具来校正时间。
* * *
### 2、[安装docker](https://docs.docker.com/install/linux/docker-ce/centos/#set-up-the-repository)  docker-ce-17.12.1.ce-1.el7.centos.x86_64
```
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
sudo yum install -y yum-utils device-mapper-persistent-data lvm2                  
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum-config-manager --enable docker-ce-edge
sudo yum search kubeadm --showduplicates
sudo yum install -y docker-ce-17.12.1.ce-1.el7.centos.x86_64
```
* docker国内加速
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xyfklydc.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
systemctl enable docker && systemctl start docker && systemctl status docker
```
***
### 3、设置环境变量
```
sed -i  '$a\export NODE_NAME='"$(hostnamectl --static)" ~/.bash_profile
sed -i  '$a\export NODE_IP='"$(ip addr | grep enp0s8 | grep inet | awk -F '[ /]+' '{print $3}')"'  # 当前部署的机器 IP' ~/.bash_profile
sed -i  '$a\export NODE_IPS="172.11.7.23,172.11.7.24,172.11.7.25" #etcd集群所有机器 IP' ~/.bash_profile
sed -i  '$a\export ETCD_NODES=fefa.master=https://172.11.7.24:2380,fefa-master-2=https://172.11.7.23:2380,fefa.master3=https://172.11.7.25:2380 #etcd 集群间通信的IP和端口' ~/.bash_profile
source ~/.bash_profile
```
- - -
### 4、安装keepalived
    yum install -y keepalived
```
cat >/etc/keepalived/keepalived.conf  <<EOF
global_defs {
   router_id $(hostnamectl --static)
}
vrrp_script CheckK8sMaster {
    script "curl -k https://172.11.7.22:6443"
    interval 3
    timeout 9
    fall 2
    rise 2
}
vrrp_instance VI_1 {
    state MASTER
    interface enp0s8
    virtual_router_id 61
    # 主节点权重最高 依次减少
    priority 120
    advert_int 1
    #修改为本地IP 
    mcast_src_ip $(ip addr | grep enp0s8 | grep inet | awk -F '[ /]+' '{print $3}')
    nopreempt
    authentication {
        auth_type PASS
        auth_pass awzhXylxy.T
    }
    unicast_peer {
      172.11.7.23
      172.11.7.24
    }
    virtual_ipaddress {
        172.11.7.22/24
    }
    track_script {
        CheckK8sMaster
    }
}
EOF
```
    systemctl enable keepalived && systemctl start keepalived && systemctl status keepalived
---
### 5、生成证书
* 安装cfssl
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
chmod +x cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```
* 制作证书
```
cat >  ca-config.json <<EOF
{
"signing": {
"default": {
  "expiry": "8760h"
},
"profiles": {
  "kubernetes-Soulmate": {
    "usages": [
        "signing",
        "key encipherment",
        "server auth",
        "client auth"
    ],
    "expiry": "8760h"
  }
}
}
}
EOF

cat >  ca-csr.json <<EOF
{
"CN": "kubernetes-Soulmate",
"key": {
"algo": "rsa",
"size": 2048
},
"names": [
{
  "C": "CN",
  "ST": "shanghai",
  "L": "shanghai",
  "O": "k8s",
  "OU": "System"
}
]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

cat > etcd-csr.json <<EOF
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "172.11.7.23",
    "172.11.7.24",
    "172.11.7.25"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "shanghai",
      "L": "shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes-Soulmate etcd-csr.json | cfssljson -bare etcd
  
mkdir -p /etc/etcd/ssl
cp etcd.pem etcd-key.pem  ca.pem /etc/etcd/ssl/
```
----
### 6、安装[etcd](https://github.com/etcd-io/etcd/releases)
```
wget https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz
tar -xvf etcd-v3.3.10-linux-amd64.tar.gz
mv etcd-v3.3.10-linux-amd64/etcd* /usr/local/bin
mkdir -p /var/lib/etcd

echo "
  [Unit]
  Description=Etcd Server
  After=network.target
  After=network-online.target
  Wants=network-online.target
  Documentation=https://github.com/coreos
  
  [Service]
  Type=notify
  WorkingDirectory=/var/lib/etcd/
  ExecStart=/usr/local/bin/etcd \\
  --name=${NODE_NAME} \\
  --cert-file=/etc/etcd/ssl/etcd.pem \\
  --key-file=/etc/etcd/ssl/etcd-key.pem \\
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \\
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \\
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \\
  --initial-advertise-peer-urls=https://${NODE_IP}:2380 \\
  --listen-peer-urls=https://${NODE_IP}:2380 \\
  --listen-client-urls=https://${NODE_IP}:2379,http://127.0.0.1:2379 \\
  --advertise-client-urls=https://${NODE_IP}:2379 \\
  --initial-cluster-token=etcd-cluster-0 \\
  --initial-cluster=${ETCD_NODES} \\
  --initial-cluster-state=new \\
  --data-dir=/var/lib/etcd
  Restart=on-failure
  RestartSec=5
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
"> etcd.service 

mv etcd.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable etcd && systemctl start etcd && systemctl status etcd
```
* 验证服务
```
etcdctl \
  --endpoints="https://172.11.7.23:2379,https://172.11.7.24:2379,https://172.11.7.25:2379"  \
  --ca-file=/etc/etcd/ssl/ca.pem \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  cluster-health

etcdctl   --endpoints=https://${NODE_IP}:2379 --ca-file=/etc/etcd/ssl/ca.pem   --cert-file=/etc/etcd/ssl/etcd.pem   --key-file=/etc/etcd/ssl/etcd-key.pem   cluster-health  
```
* 启动时报错 “request sent was ignored”，可能由于其他配置文件参数错误导致无法启动，但是已经在/var/lib/etcd/目录中生成了初始化文件导致，那么删除/var/lib/etcd/下文件即可
    rm -Rf /var/lib/etcd /var/lib/etcd-cluster && mkdir -p /var/lib/etcd
---
### 7、安装[kubernates](https://www.kubernetes.org.cn/3536.html)
#### 7.2 安装kubeadm kubelet kubectl
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum search kubeadm --showduplicates
yum install -y kubelet kubeadm kubectl kubernetes-cni
yum install -y kubeadm-1.10.0-0.x86_64 kubectl-1.10.0-0.x86_64 kubelet-1.10.0-0.x86_64 

```
#### 7.2 配置
* KUBELET_CGROUP_ARGS=--cgroup-driver=systemd为KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs
```
cat > /etc/systemd/system/kubelet.service.d/20-pod-infra-image.conf <<EOF
[Service]
Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/osoulmate/pause-amd64:3.0"
EOF

systemctl daemon-reload && systemctl enable kubelet && systemctl restart kubelet

cat <<EOF > config.yaml 
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
etcd:
  endpoints:
  - https://172.11.7.23:2379
  - https://172.11.7.24:2379
  - https://172.11.7.25:2379
  caFile: /etc/etcd/ssl/ca.pem 
  certFile: /etc/etcd/ssl/etcd.pem 
  keyFile: /etc/etcd/ssl/etcd-key.pem
  dataDir: /var/lib/etcd
networking:
  podSubnet: 10.244.0.0/16
kubernetesVersion: v1.10.0
api:
  advertiseAddress: "172.11.7.22"
token: "b33a99.a244ef77531e4354"
tokenTTL: "0s"
apiServerCertSANs:
- fefa.master
- fefa-master-2
- fefa.master3
- 172.11.7.23
- 172.11.7.24
- 172.11.7.25
- 172.11.7.22
featureGates:
  CoreDNS: true
imageRepository: "registry.cn-hangzhou.aliyuncs.com/kubernetes1112images"
EOF

kubeadm init --config config.yaml 
# 失败后可进行重置
kubeadm reset 
```
#### 7.3 安装网络组件podnetwork,[kube-router](https://github.com/cloudnativelabs/kube-router/blob/master/daemonset/kubeadm-kuberouter.yaml)
```
wget https://github.com/cloudnativelabs/kube-router/blob/master/daemonset/kubeadm-kuberouter.yaml
kubectl apply -f kubeadm-kuberouter.yaml
```

#### 7.4 初始化其他master节点
```
#拷贝pki 证书
mkdir -p /etc/kubernetes/pki
scp -r root@172.11.7.23:/etc/kubernetes/pki /etc/kubernetes

#拷贝初始化配置
scp -r root@172.11.7.23:/home/tools/config.yaml /etc/kubernetes/config.yaml
kubeadm init --config /etc/kubernetes/config.yaml 
```

#### 7.5 测试高可用，关闭其中一台服务，观察其他服务
* 验证主备VIP切换

    systemctl status keepalived
+ 验证集群高可用   
    while true; do  sleep 1; kubectl get node;date; done

结果：



NAME | STATUS | ROLES | AGE | VERSION
------ | ------ | ------ | ------ | ------ |
fefa-master-2 | Ready | master | 1h | v1.10.0
|fefa.master | NotReady | master | 13m | v1.10.0 |

#### 7.6 安装dashboard
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```