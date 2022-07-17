# Docker 镜像



本节的示例，基本都是基于项目: [cloud-tool](https://github.com/Nailcui/cloud-tool) 来进行演示的



## Docker 镜像相关的操作

查看本地有哪些镜像

```
docker images
```

搜索镜像

```
╰─$ docker search alpine
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
alpine                            A minimal Docker image based on Alpine Linux…   8998      [OK]
alpine/git                        A  simple git container running in alpine li…   197                  [OK]
```

拉取镜像

```
docker image pull alpine:latest
```

构建镜像

```
docker build . -t naildocker/cloud-tool:0.1.1
```

推送镜像到官方仓库

```
docker push naildocker/cloud-tool:0.1.1
```



## Docker 镜像里有什么

需要了解的知识: `rootfs`、`chroot`、`pivot_root`

- pivot_root是一个系统调用，主要功能是去改变当前的 root文件系统。 pivot_root可以将当 前进程的 root 文件系统移动到 put_old 文件夹中，然后使 new_root 成为新的 root 文件系统。



我们现在有个已经启动的容器: `bec7145e843f   b331d7606e08  ./cloud-tool` 

获取这个容器的 rootfs

```
[root@dev-k8s-node01 ~]# docker export -o cloud-tool.tar bec7145e843f
[root@dev-k8s-node01 ~]# ll
total 12356
-rw------- 1 root root 12650496 2022-07-17 12:35:03 cloud-tool.tar
```

看下生成的`cloud-tool.tar` 里有什么

```
[root@dev-k8s-node01 ~]# mkdir cloud-tool
[root@dev-k8s-node01 ~]# tar -xvf cloud-tool.tar -C cloud-tool/
```

```
[root@dev-k8s-node01 cloud-tool-image]# cd cloud-tool/
[root@dev-k8s-node01 cloud-tool]# ll
total 72
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 bin
drwxr-xr-x  2 root root 4096 2022-07-17 12:01:08 build
drwxr-xr-x  4 root root 4096 2022-07-17 12:08:43 dev
drwxr-xr-x 15 root root 4096 2022-07-17 12:08:43 etc
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 home
drwxr-xr-x  7 root root 4096 2021-04-14 18:25:48 lib
drwxr-xr-x  5 root root 4096 2021-04-14 18:25:48 media
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 mnt
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 opt
dr-xr-xr-x  2 root root 4096 2021-04-14 18:25:48 proc
drwx------  2 root root 4096 2021-04-14 18:25:48 root
drwxr-xr-x  3 root root 4096 2022-07-17 12:08:43 run
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 sbin
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 srv
drwxr-xr-x  2 root root 4096 2021-04-14 18:25:48 sys
drwxrwxrwt  2 root root 4096 2021-04-14 18:25:48 tmp
drwxr-xr-x  7 root root 4096 2021-04-14 18:25:48 usr
drwxr-xr-x 12 root root 4096 2021-04-14 18:25:48 var
```

可以看到这基本上就是一个我们平常使用的linux系统的完成的文件



## Dockerfile 的编写



## Docker镜像最佳实践

示例 `Dockerfile`：

```
FROM golang:1.16-alpine AS builder
WORKDIR /build
COPY . /build
RUN GOPROXY=https://goproxy.cn CGO_ENABLED=0 go build -ldflags="-w -s"

FROM alpine AS runner
WORKDIR /build
COPY --from=builder /build/cloud-tool .
EXPOSE 80
CMD ["./cloud-tool"]
```

##### 1、多阶段构建

上面例子中，分了2个阶段：编译阶段、生成目标镜像阶段；因为`go`语言的项目，直接生成可执行文件，所以最后运行环境不需要go语言相关的工具链，所以可以将`build` 阶段单独用一个含有go环境的镜像来作为基础镜像

##### 2、在满足需求的情况下使用尽量小的镜像

第一个阶段必须用go环境，使用了 `golang:1.16-alpine` 的镜像，大小约300M；

第二个阶段，直接用了 alpine镜像，大小约5.6M；



## Docker镜像大小优化

