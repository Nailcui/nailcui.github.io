[grpc 官方 quick start](https://grpc.io/docs/languages/go/quickstart/)

[protocol-buffers doc](https://developers.google.com/protocol-buffers/docs/proto3#simple)



在本节中，我们先使用 k8s 的`service` 作为服务端负载均衡方式，实现了功能；

之后用 `Headless service` 配合 `go-grpc` 的`dns` `roundrobin` 来达到了真正负载均衡的效果



步骤：

- 安装相关的工具，主要是和proto相关的代码生成工具
- 编写 proto
- 生成代码
- 编写client、server代码
- 打包 build
- 打镜像
- 部署到k8s



### 安装相关工具

安装 proto 生成工具

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```



安装 protoc 编译工具

```
# 官方安装文档: https://grpc.io/docs/protoc-installation/
# mac
brew install protobuf
```



### 编写 proto 协议文件

```protobuf
syntax = "proto3";

option go_package = "go-example/grpc-example";

service HostnameService {
  rpc Hostname (HostnameRequest) returns (HostnameResponse);
}

message HostnameRequest {
  string hostname = 1;
}

message HostnameResponse {
  string hostname = 1;
}

```



### 生成代码

```
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative  hostname.proto
```



### 编写 server 逻辑

```go
package main

import (
	"context"
	"log"
	"net"
	"os"

	"go-example/grpc-example"
	"google.golang.org/grpc"
)

type server struct {
	grpc_example.UnimplementedHostnameServiceServer
}

func (s server) Hostname(ctx context.Context, request *grpc_example.HostnameRequest) (*grpc_example.HostnameResponse, error) {
	hostname, err := os.Hostname()
	return &grpc_example.HostnameResponse{Hostname: hostname}, err
}

func main() {
	lis, err := net.Listen("tcp", ":9090")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

    s := grpc.NewServer()
	grpc_example.RegisterHostnameServiceServer(s, &server{})

    log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

```



### 编写 client 逻辑

```go
package main

import (
	"log"
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
	"go-example/grpc-example"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

func main() {
	router := gin.Default()
	router.GET("/rpc", handler)
	_ = router.Run(":80")
}

func handler(c *gin.Context) {
	hostname, err := os.Hostname()
	if err != nil {
		c.String(http.StatusBadRequest, err.Error())
		return
	}

    conn, err := grpc.Dial("grpc-example-service.default.svc.cluster.local:9090", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer func () {
		_ =  conn.Close()
	}()

    client := grpc_example.NewHostnameServiceClient(conn)
	response, err := client.Hostname(c, &grpc_example.HostnameRequest{Hostname: hostname})
	if err != nil {
		c.String(http.StatusBadRequest, err.Error())
		return
	}
	c.String(http.StatusOK, response.Hostname)
}

```



### build

```
GOOS=linux GOARCH=amd64 go build grpc-client.go
GOOS=linux GOARCH=amd64 go build grpc-server.go
```



### docker build

```
docker build . -t naildocker/grpc-example-client:0.0.1
docker build . -t naildocker/grpc-example-server:0.0.1
```



### docker push

```
docker push naildocker/grpc-example-client:0.0.1
docker push naildocker/grpc-example-server:0.0.1
```



### 部署到k8s

#### grpc-example-client.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grpc-example-client
  labels:
    env: DEV
    app: grpc-example-client
spec:
  selector:
    matchLabels:
      app: grpc-example-client
  replicas: 2
  template:
    metadata:
      labels:
        app: grpc-example-client
    spec:
      containers:
        - name: grpc-example-client
          image: naildocker/grpc-example-client:0.0.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
```



#### grpc-example-server.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grpc-example-server
  labels:
    env: DEV
    app: grpc-example-server
spec:
  selector:
    matchLabels:
      app: grpc-example-server
  replicas: 2
  template:
    metadata:
      labels:
        app: grpc-example-server
    spec:
      containers:
        - name: grpc-example-server
          image: naildocker/grpc-example-server:0.0.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 9090
```



#### grpc-example-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grpc-example-service
spec:
  type: LoadBalancer
  selector:
    app: grpc-example-server
  ports:
    - port: 9090
      targetPort: 9090
```

#### grpc-example-client-ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grpc-example-client-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # 开启use-regex，启用path的正则匹配
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    # 定义域名
    - host: grpc-example-client
      http:
        paths:
        # 不同path转发到不同端口
          - path: /
            backend:
              serviceName: grpc-example-client
              servicePort: 80
```



### 测试

```shell
[root@dev-k8s-node01 grpc-example]# curl grpc-example-client:80/rpc
grpc-example-server-58c6676c99-l6767[root@dev-k8s-node01 grpc-example]#
[root@dev-k8s-node01 grpc-example]# curl grpc-example-client:80/rpc
grpc-example-server-58c6676c99-rtskd[root@dev-k8s-node01 grpc-example]#
```



----

使用 grpc的负载均衡

#### go client 代码

```go
package main

import (
	"flag"
	"fmt"
	"google.golang.org/grpc/balancer/roundrobin"
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
	"go-example/grpc-example"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

var (
	conn *grpc.ClientConn
	client grpc_example.HostnameServiceClient
)

var (
	addr = flag.String("addr", "grpc-example-service.default.svc.cluster.local:9090", "the address to connect ")
)

func init() {
	flag.Parse()
	conn, _ = grpc.Dial(*addr,
		grpc.WithDefaultServiceConfig(fmt.Sprintf(`{"LoadBalancingPolicy":"%s"}`, roundrobin.Name)),
		grpc.WithTransportCredentials(insecure.NewCredentials()))
	client = grpc_example.NewHostnameServiceClient(conn)
}

func main() {
	router := gin.Default()
	router.GET("/rpc", handler)
	_ = router.Run(":80")
}

func handler(c *gin.Context) {
	hostname, err := os.Hostname()
	if err != nil {
		c.String(http.StatusBadRequest, err.Error())
		return
	}
	response, err := client.Hostname(c, &grpc_example.HostnameRequest{Hostname: hostname})
	if err != nil {
		c.String(http.StatusBadRequest, err.Error())
		return
	}
	c.String(http.StatusOK, response.Hostname)
}

```



#### Headless service 配置

```
apiVersion: v1
kind: Service
metadata:
  name: grpc-example-service
spec:
  clusterIP: None
  selector:
    app: grpc-example-server
  ports:
    - port: 9090
      targetPort: 9090

```

