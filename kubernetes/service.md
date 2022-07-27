# service

相关资源：

- [官方文档](https://kubernetes.io/docs/concepts/services-networking/service)





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

当不需要负载均衡和service ip 的时候，可以指定为`Headless`模式，此时DNS指向所有的backend

##### DNS

“普通” Service（除了无头 Service）会以 `my-svc.my-namespace.svc.cluster-domain.example` 这种名字的形式被分配一个 DNS A 或 AAAA 记录，取决于 Service 的 IP 协议族。 该名称会解析成对应 Service 的集群 IP。

“无头（Headless）” Service （没有集群 IP）也会以 `my-svc.my-namespace.svc.cluster-domain.example` 这种名字的形式被指派一个 DNS A 或 AAAA 记录， 具体取决于 Service 的 IP 协议族。 与普通 Service 不同，这一记录会被解析成对应 Service 所选择的 Pod IP 的集合。 客户端要能够使用这组 IP，或者使用标准的轮转策略从这组 IP 中进行选择。



#### 2、NodePort

将端口暴露到node上



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

