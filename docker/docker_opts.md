#### 容器操作

```sh
# 创建容器
docker create busybox
# 运行容器
docker start busybox

# 直接运行容器
docker run busybox

# -it 启动tty，进入交互模式
# -p 指定端口映射
# --name 指定名称
# -v 卷映射
# -d 在后台运行: docker run -d busybox top -b
docker run -it -p 8080:80 --name busybox busybox /bin/sh

# 停止
docker stop <container-id>

# 删除
docker rm <container-id>

docker top <container-id>

# 进入容器内部操作
docker exec -it <container-id> /bin/sh

# 查看日志
# -f 滚动输出
docker logs -f <container-id>

# 复制文件
docker cp c484b111a7e4:/etc/hosts .

```



#### 容器管理

```sh
# 输出所有容器
# -a 所有，默认只输出正在运行中的容器
docker ps -a
```



#### 镜像操作

```sh
# 列出当前镜像
docker images

# 搜索镜像
docker search busybox

# 下载busybox的镜像
docker pull busybox

# 删除镜像
docker rmi busybox

# 构建镜像
docker build -t naildocker/cloud-tool:0.1
```



#### 导出容器的rootfs

```sh
docker export $(docker create busybox) | tar -C rootfs -xvf -
```

