# Kubernetes Replica Sets

ReplicaSet（RS）是Replication Controller（RC）的升级版本。ReplicaSet 和  Replication Controller之间的唯一区别是对选择器的支持。ReplicaSet支持labels user guide中描述的set-based选择器要求， 而Replication Controller仅支持equality-based的选择器要求。

## 如何使用ReplicaSet

大多数kubectl 支持Replication Controller 命令的也支持ReplicaSets。rolling-update命令除外，如果要使用rolling-update，请使用Deployments来实现。

虽然ReplicaSets可以独立使用，但它主要被 Deployments用作pod 机制的创建、删除和更新。当使用Deployment时，你不必担心创建pod的ReplicaSets，因为可以通过Deployment实现管理ReplicaSets。

## 何时使用ReplicaSet

ReplicaSet能确保运行指定数量的pod。然而，Deployment 是一个更高层次的概念，它能管理ReplicaSets，并提供对pod的更新等功能。因此，我们建议你使用Deployment来管理ReplicaSets，除非你需要自定义更新编排。

这意味着你可能永远不需要操作ReplicaSet对象，而是使用Deployment替代管理 。

```
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: frontend
  # these labels can be applied automatically
  # from the labels in the pod template if not set
  # labels:
    # app: guestbook
    # tier: frontend
spec:
  # this replicas value is default
  # modify it according to your case
  replicas: 3
  # selector can be applied automatically
  # from the labels in the pod template if not set,
  # but we are specifying the selector here to
  # demonstrate its usage.
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80

```