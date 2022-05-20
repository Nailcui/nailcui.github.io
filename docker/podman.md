# podman

## 什么是 podman？

> ### Podman is a daemonless container engine for developing, managing, and running OCI Containers on your Linux System

`Podman` 原来是 [CRI-O](https://github.com/kubernetes-incubator/cri-o) 项目的一部分，后来被分离成一个单独的项目叫 [libpod](https://github.com/containers/libpod)。Podman 的使用体验和 `Docker` 类似。



## podman 和 docker 的区别

哈哈: `alias docker='podman'` 

从官网描述中，即可看出来和docker的最大区别: `daemonless container engine` 

### 架构上的区别：

Podman 没有 daemon。

以前使用 Docker CLI 的时候，Docker CLI 会通过 gRPC API 去跟 Docker Engine 说「我要启动一个容器」，然后 Docker Engine 才会通过 OCI Container runtime（默认是 `runc`）来启动一个容器。

这就意味着容器的进程不可能是 Docker CLI 的子进程，而是 Docker Engine 的子进程。



Podman 比较简单粗暴，它不使用 Daemon，而是直接通过 OCI runtime（默认也是 `runc`）来启动容器，所以容器的进程是 podman 的子进程。

这比较像 Linux 的 `fork/exec` 模型，而 Docker 采用的是 `C/S`（客户端/服务器）模型。与 C/S 模型相比，`fork/exec` 模型有很多优势，比如：

- 系统管理员可以知道某个容器进程到底是谁启动的。
- 如果利用 `cgroup` 对 podman 做一些限制，那么所有创建的容器都会被限制。
- **SD_NOTIFY** : 如果将 podman 命令放入 `systemd` 单元文件中，容器进程可以通过 podman 返回通知，表明服务已准备好接收任务。
- **socket 激活** : 可以将连接的 `socket` 从 systemd 传递到 podman，并传递到容器进程以便使用它们。



### 功能上的区别

- 支持普通用户运行 rootless 容器（kernel >= 4.9.0 ）