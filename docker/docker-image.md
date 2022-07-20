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



拉取下来的镜像怎么查看这个镜像是怎么构建的呢？

```shell
[root@dev-k8s-node01 ~]# docker image history naildocker/cloud-tool:0.1.1
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
b331d7606e08   2 days ago      CMD ["./cloud-tool"]                            0B        buildkit.dockerfile.v0
<missing>      2 days ago      EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      2 days ago      COPY /build/cloud-tool . # buildkit             6.77MB    buildkit.dockerfile.v0
<missing>      2 days ago      WORKDIR /build                                  0B        buildkit.dockerfile.v0
<missing>      15 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      15 months ago   /bin/sh -c #(nop) ADD file:8ec69d882e7f29f06…   5.61MB
```



构建镜像

```
docker build . -t naildocker/cloud-tool:0.1.1
```



推送镜像到官方仓库

```
docker push naildocker/cloud-tool:0.1.1
```



查看镜像的详细信息

```shell
[root@dev-k8s-node01 ~]# docker image inspect naildocker/cloud-tool:0.1.1
[
    {
        "Id": "sha256:b331d7606e08ea6b747e6592bde1f4ee2f87843930c4880fb31f2776f8cea3d2",
        "RepoTags": [
            "naildocker/cloud-tool:0.1.1"
        ],
        "RepoDigests": [
            "naildocker/cloud-tool@sha256:7e083939007bc0f9bd3f2c6d00a01e68fa63896767930078e8084eb3d7fded8d"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2022-07-17T04:01:08.543742085Z",
        "Container": "",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": null,
            "Cmd": null,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "./cloud-tool"
            ],
            "ArgsEscaped": true,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "/build",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 12379750,
        "VirtualSize": 12379750,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/effbee484f064347ef0d6c5db51b8307cf2da00599d68bd84f0a7934cd974387/diff:/var/lib/docker/overlay2/5dc9c3483a6bdc4d8118e8930065001aeae1e8d6b5a54df8b7b81427a995a57a/diff",
                "MergedDir": "/var/lib/docker/overlay2/9b8c537a684fc6d3df3aa165bdd4e1ee964c61eb251c7cb4e62ddc017eb53cfd/merged",
                "UpperDir": "/var/lib/docker/overlay2/9b8c537a684fc6d3df3aa165bdd4e1ee964c61eb251c7cb4e62ddc017eb53cfd/diff",
                "WorkDir": "/var/lib/docker/overlay2/9b8c537a684fc6d3df3aa165bdd4e1ee964c61eb251c7cb4e62ddc017eb53cfd/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:b2d5eeeaba3a22b9b8aa97261957974a6bd65274ebd43e1d81d0a7b8b752b116",
                "sha256:2c520f6588831d097b9d3a54887e8869618bc70ec57cf4b1f880045f5f3daf97",
                "sha256:17d1425b31125389a90152c131bd17cb134f6080bd352c0133f3bd042cba1b8c"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```



清理本地无用镜像

```shell
docker image prune
```

```shell
[root@dev-k8s-node01 ~]# docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```





## Docker 镜像里有什么

需要了解的知识: `rootfs`、`chroot`、`pivot_root`、`union file system`

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

可以看到这基本上就是一个我们平常使用的linux系统的完成的文件；

这里一般是不包含操作系统内核的，只有文件系统



## Dockerfile 的编写



## Docker镜像最佳实践

示例 `Dockerfile`：

```dockerfile
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

在`alpine` 的镜像中，一些最基本的命令也没有提供，比如`curl`等用来排查问题比较常用的命令；所以正式环境要注意这些差别，选择合适的镜像

##### 明确From镜像的版本

不要使用`latest`版本，使用确定的版本；否则之后from镜像更新后，会出现不兼容的情况



## Docker镜像大小优化

