# service

相关资源：

- [官方文档](https://kubernetes.io/docs/concepts/services-networking/service)



**service**：pod的状态处于实时变化之中，为了找到这些`EndPoint`，提供了`Service`：通过DNS的方式提供服务的发现/负载均衡方案



### 最基本的资源定义

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```



### 服务类型

我们可以通过 `Kubernetes ServiceTypes` 指定所需的服务类型，默认值为ClusterIP。



#### 1、ClusterIP

每当创建一个`service`时， Kubernetes 会创建一个相应的DNS（DNS指向service的ip）：相同命名空间下可以直接通过 `<服务名称>` 访问，跨命名空间需要：`<服务名称>.<名字空间名称>.svc.cluster.local`

##### Headless Services

```yaml
spec:
    clusterIP: None
```

```shell
# CLUSTER-IP 是 None
[root@dev-k8s grpc-example]# kubectl get service
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
grpc-example-service   ClusterIP   None            <none>        9090/TCP   2m53s
```



指定为`Headless`模式，此时DNS指向所有的backend，用查看dns记录：

```bash
[root@dev-k8s-node01 grpc-example]# kubectl exec grpc-example-client-98b874ddb-ff5xv -- nslookup grpc-example-service.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   grpc-example-service.default.svc.cluster.local
Address: 192.168.128.21
Name:   grpc-example-service.default.svc.cluster.local
Address: 192.168.128.20
```



可以借此实现客户端负载均衡，比如在`go-grpc`中，如下代码：

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
	addr = flag.String("addr", "dns:///grpc-example-service.default.svc.cluster.local:9090", "the address to connect ")
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



##### DNS

“普通” Service（除了无头 Service）会以 `my-svc.my-namespace.svc.cluster-domain.example` 这种名字的形式被分配一个 DNS A 或 AAAA 记录，取决于 Service 的 IP 协议族。 该名称会解析成对应 Service 的集群 IP。

“无头（Headless）” Service （没有集群 IP）也会以 `my-svc.my-namespace.svc.cluster-domain.example` 这种名字的形式被指派一个 DNS A 或 AAAA 记录， 具体取决于 Service 的 IP 协议族。 与普通 Service 不同，这一记录会被解析成对应 Service 所选择的 Pod IP 的集合。 客户端要能够使用这组 IP，或者使用标准的轮转策略从这组 IP 中进行选择。



#### 2、NodePort

将端口暴露到node上，注意端口范围



#### 3、LoadBalancer

使用云厂商的 LoadBalancer



#### 4、ExternalName

类型为 ExternalName 的服务将服务映射到 DNS 名称，而不是典型的选择器，例如 `my-service` 或者 `cassandra`。 你可以使用 `spec.externalName` 参数指定这些服务。

例如，以下 Service 定义将 `prod` 名称空间中的 `my-service` 服务映射到 `my.database.example.com`：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```



### 实现原理

使用 iptables 处理流量具有较低的系统开销，因为流量由 Linux netfilter 处理， 而无需在用户空间和内核空间之间切换。 这种方法也可能更可靠。

IPVS 代理模式基于类似于 iptables 模式的 netfilter 挂钩函数， 但是使用哈希表作为基础数据结构，并且在内核空间中工作。 这意味着，与 iptables 模式下的 kube-proxy 相比，IPVS 模式下的 kube-proxy 重定向通信的延迟要短，并且在同步代理规则时具有更好的性能。 与其他代理模式相比，IPVS 模式还支持更高的网络流量吞吐量。IPVS 提供了更多选项来平衡后端 Pod 的流量