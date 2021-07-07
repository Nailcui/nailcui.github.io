在k8s的编程模型整体上来说类似于观察者模式，kube-apiserver负责etcd数据的维护；每个人可以编写`controller` 代码监听`kube-apiserver` 获得资源的变化事件，并做出反应；这个资源我们简单说一下，资源可以理解为一组约束条件，从表现上来看，就是1个yaml文件，里面声明了各种约束条件，比如副本数量想要达到多少，镜像想使用哪个等。

本文所说的 `controller` 是kubenetes官方自己的实现，不同的 `controller` 实现了不通的逻辑，例如：

- `Replication Controller` 是用来监听`RC` 资源，使得集群中的pod副本数保持和`RC` 中声明的数量一致
- `Node Controller` 用来监听节点的变化，对节点的上线、下线等事件做出处理
- `ResourceQuota Controller` 负责资源配额的管理
- `Namespace Controller` 处理`Namespace` 的创建、删除
- `Service Controller` 
- `Endpoint Controller` 

