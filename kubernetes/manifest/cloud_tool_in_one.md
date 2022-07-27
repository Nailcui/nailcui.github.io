```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloud-tool
spec:
  selector:
    matchLabels:
      app: cloud-tool
  replicas: 3
  template:
    metadata:
      labels:
        app: cloud-tool
    spec:
      containers:
      - name: cloud-tool
        image: naildocker/cloud-tool:0.1.1
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: cloud-tool-svc
spec:
  selector:
    app: cloud-tool
  ports:
  - port: 80
    targetPort: 80
---

# apiVersion: networking.k8s.io/v1beta1 # version 1.14 to 1.18
apiVersion: networking.k8s.io/v1 # version 1.19+
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  rules:
  - host: ct.baidu.com
    http:
      paths:
      - path: /
        backend:
          serviceName: cloud-tool-svc
          servicePort: 80

```

测试 ingress 是否好用：

```
curl localhost -H 'Host:ct.baidu.com' # 我们没有配置域名，只好修改hosts，然后用-H指定Host
ok
```

