# Kubernetes Namespaces

Kubernetes可以使用Namespaces（命名空间）创建多个虚拟集群。
## 何时使用多个Namespaces

当团队或项目中具有许多用户时，可以考虑使用Namespace来区分，a如果是少量用户集群，可以不需要考虑使用Namespace，如果需要它们提供特殊性质时，可以开始使用Namespace。

Namespace为名称提供了一个范围。资源的Names在Namespace中具有唯一性。

Namespace是一种将集群资源划分为多个用途(通过 resource quota)的方法。

在未来的Kubernetes版本中，默认情况下，相同Namespace中的对象将具有相同的访问控制策略。

对于稍微不同的资源没必要使用多个Namespace来划分，例如同意软件的不同版本，可以使用labels(标签)来区分同一Namespace中的资源。

## 使用 Namespaces
* 创建
```
# 命令行
kubectl create namespace new-namespace

# 文件描述
cat > my-namespace.yaml << EOF
apiVersion: v1
kind: Namespace
metadata:
  name: new-namespace
EOF

kubectl create -f ./my-namespace.yaml
```
> 注意：命名空间名称满足正则表达式[a-z0-9]([-a-z0-9]*[a-z0-9])?,最大长度为63位

* 删除

```
kubectl delete namespaces new-namespace
```

注意：

1. 删除一个namespace会自动删除所有属于该namespace的资源。
2. default和kube-system命名空间不可删除。
3. PersistentVolumes是不属于任何namespace的，但PersistentVolumeClaim是属于某个特定namespace的。
4. Events是否属于namespace取决于产生events的对象。

* 查看

```
kubectl get namespaces
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
```

Kubernetes从两个初始的Namespace开始：

1. default
2. kube-system 由Kubernetes系统创建的对象的Namespace

## Setting the namespace for a request

要临时设置Request的Namespace，请使用--namespace 标志。
```
kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
$ kubectl --namespace=<insert-namespace-name-here> get pods
```

## Setting the namespace preference

可以使用kubectl命令创建的Namespace可以永久保存在context中。

```
$ kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
# Validate it
$ kubectl config view | grep namespace:
```

## 所有对象都在Namespace中?

大多数Kubernetes资源（例如pod、services、replication controllers或其他）都在某些Namespace中，但Namespace资源本身并不在Namespace中。而低级别资源（如Node和persistentVolumes）不在任何Namespace中。Events是一个例外：它们可能有也可能没有Namespace，具体取决于Events的对象。