使用场景：

- 一个集群3个节点，可以提前知道各个节点访问时候的域名
- 持久的存储



和 Deployment 的一些不同点

- 可以固定 主机名
- 可以固定网络标志（域名），ip是不固定的，不过厂商有固定ip的实现
- 扩容时，从小到大；缩容时，从大到小
- 滚动发布时，先下线，再上线



之前的 `cloud-tool`，用 `StatefulSet`部署一下

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sts-cloud-tool-svc
  labels:
    app: sts-cloud-tool
spec:
  ports:
  - port: 80
    name: web
  selector:
    app: sts-cloud-tool
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-cloud-tool
spec:
  selector:
    matchLabels:
      app: sts-cloud-tool
  serviceName: sts-cloud-tool
  replicas: 1
  template:
    metadata:
      labels:
        app: sts-cloud-tool
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: cloud-tool
        image: naildocker/cloud-tool:0.1.2
        ports:
        - containerPort: 80
          name: web

```

