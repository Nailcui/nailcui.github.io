# pod



> Pod，是Kubernetes项 目中最小的API对象，是Kubernetes项目的原子调度单位。

### 延伸阅读：

[容器设计模式](https://blog.csdn.net/qq_43762191/article/details/124603595)

[容器设计模式 - 为什么我们需要Pod？- 知乎](https://zhuanlan.zhihu.com/p/416586456)



### 怎么理解pod

Pod 的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离方面， 即用来隔离 Docker 容器的技术。 在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

就 Docker 概念的术语而言，Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器。

**如果将 k8s 类比为操作系统，那 pod就相当于 “进程组”。**



### 为什么Pod（含有一个或者多个Container）是最小的部署单元，而不能直接是容器？

Pod是一组共享生命周期，并部署在同一个节点的容器的组合，他们可以通过共享的volume/network和IPC来进行通讯。

之所以不是一个单一容器，而是多个容器来完成特定功能的原因在于：

1、这些容器要完成的职责不同，根据单一职责（single responsibility）的原则，他们应该属于不同的组件；

2、其次因为职责不同，维护他们的team也不同，迭代周期也不一样；

3、最后其中一些容器是可以被复用在其他的环境中的。



所以从“解耦”和“复用”的设计原则出发，Kubernetes通过增加一个虚拟层即POD，给系统设计带来了极大的灵活性，同时也产生了多种设计模式。即在一个POD中除了抗流量完成业务的容器外，还存在其他的辅助容器，可以分为两类：

- Init Container 

- Sidecar container。


