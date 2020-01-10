# Kubernetes Names

Kubernetes REST API中的所有对象都用Name和UID来明确地标识。

> 对于非唯一用户提供的属性，Kubernetes提供[labels]()和[annotations]()。

## Name
Name在一个对象中同一时间只能拥有单个Name，如果对象被删除，也可以使用相同Name创建新的对象，Name用于在资源引用URL中的对象，例如/api/v1/pods/some-name。通常情况，Kubernetes资源的Name能有最长到253个字符（包括数字字符、-和.），但某些资源可能有更具体的限制条件，具体情况可以参考：[标识符设计文档]。

## UIDs

UIDs是由Kubernetes生成的，在Kubernetes集群的整个生命周期中创建的每个对象都有不同的UID（即它们在空间和时间上是唯一的）。