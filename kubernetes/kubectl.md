# kubectl

`kubectl`是kubernetes的命令行工具，其也是通过和`kube-apiserver`交互来实现相关功能。

`kubectl` 和k8s控制面交互，这些密钥信息是哪里来的呢，默认是: `~/.kube/config`



```shell
kubectl [command] [TYPE] [NAME] [flags]
```

kubectl支持很多命令，另外`kubectl`也支持扩展，可以添加自己的子命令插件





### kubectl 插件

官方提供了一个插件的包管理工具[krew](https://github.com/kubernetes-sigs/krew)；在这里我们可以方便的搜索、安装别人开发好的插件

我们也可以实现自己的插件，这里我们添加一个`hello`的插件，最终可以实现如下效果：

```shell
$ kubectl hello
hello world
```



#### 第一步：添加可执行文件

可执行文件可以是：shell脚本、其他语言脚本、二进制文件

我们已一个`shell`脚本为例：

```shell
# 编写脚本文件kubectl-hello，命名有要求：必须已 kubectl- 开头
# 内容：

#!/bin/sh

# 这个插件打印单词 "hello world"
echo "hello world"
```



#### 第二步：添加到PATH&添加执行权限

```shell
# 赋予执行权限
sudo chmod a+x ./kubectl-hello

# 并将其移动到 PATH 中的某个位置
sudo mv ./kubectl-hello /usr/local/bin
sudo chown root:root /usr/local/bin
```





### kubectl 的常见命令

```shell
kubectl apply -f example-service.yaml
kubectl apply -f <directory>

kubectl get pods
kubectl get pods -o wide

kubectl describe pods <rc-name>

kubectl delete -f pod.yaml
```



```shell
kubectl exec <pod-name> -- date
kubectl exec -ti <pod-name> -- /bin/bash
```

```shell
kubectl logs <pod-name>
kubectl logs -f <pod-name>
```





### k8s 资源管理方式

- **命令式对象管理**：直接使用命令去操作kubernetes资源

```shell
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- **命令式对象配置**：通过命令配置和配置文件去操作kubernetes资源

```shell
kubectl create/patch -f nginx-pod.yaml
```

- 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源

```shell
kubectl apply -f nginx-pod.yaml
```

| 类型           | 操作对象   | 适用环境 | 优点           | 缺点                             |
| -------------- | ---------- | -------- | -------------- | -------------------------------- |
| 命令式对象管理 | 对象       | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件       | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
| 声明式对象配置 | 文件、目录 | 开发     | 支持目录操作   | 意外情况下难以调试               |


