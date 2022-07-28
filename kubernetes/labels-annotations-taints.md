看下我们之前的一个pod，用`yaml`格式输出

```bash
[root@dev-k8s-node01 cloudtool]# kubectl get pod cloud-tool-686c6d8c8f-8c2s7 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:  # 注解
    cni.projectcalico.org/containerID: 91b31699c098a66757074ae511efd3078c88c4896954f7c2a80d8f2125086280
    cni.projectcalico.org/podIP: 192.168.128.8/32
    cni.projectcalico.org/podIPs: 192.168.128.8/32
  creationTimestamp: "2022-07-22T14:02:39Z"
  generateName: cloud-tool-686c6d8c8f-
  labels:  # 标签
    app: cloud-tool
    pod-template-hash: 686c6d8c8f
......
```



Annotation是用户任意定义的“附加”信息，以便于外部工具进行查找，比如`alpha.istio.io/sidecar`注解就是用来控制是否自动向 pod 中注入 sidecar 的



从API上观察：

```go
type DeploymentInterface interface {
	List(ctx context.Context, opts metav1.ListOptions) (*v1.DeploymentList, error)
}

type ListOptions struct {
	LabelSelector string `json:"labelSelector,omitempty" protobuf:"bytes,1,opt,name=labelSelector"`
}

```

从API上我们可以看到他们被设计的区别：

- 标签可以用来查找资源，而注解不可以



### 标签

标签能够支持高效的查询和监听操作，对于用户界面和命令行是很理想的。

[官方文档](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)

```
kubectl label nodes <node-name> <label-key>=<label-value> 

kubectl get nodes --show-labels
kubectl get pods -A --show-labels
```

查看pod并显示标签

```
[root@dev-k8s-node01 cloudtool]# kubectl get pods --show-labels
NAME                                   READY   STATUS    RESTARTS   AGE     LABELS
cloud-tool-686c6d8c8f-qwz68            1/1     Running   0          5d15h   app=cloud-tool,pod-template-hash=686c6d8c8f
grpc-example-client-98b874ddb-ff5xv    1/1     Running   0          23m     app=grpc-example-client,pod-template-hash=98b874ddb
```

添加标签

```
kubectl label pod <pod-id> key=values
```

修改标签

```
kubectl label pod <pod-id> key=values --overwrite
```

### 注解

[官方文档](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/annotations/)



注解为对象附加任意的非标识的元数据。客户端程序（例如工具和库）能够获取这些元数据信息。



### 污点

污点和容忍是用来做调度相关控制的，比如有个node上有GPU，所以我可以通过污点/容忍来控制将需要GPU的程序部署到相关node上