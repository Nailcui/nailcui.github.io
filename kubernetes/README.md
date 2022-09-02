

[理解 Kubernetes 对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/)

[Kubernetes 对象管理](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/object-management/)





### 自动扩缩容

1、官方 HPA

2、阿里 CronHPA



#### 一、HPA

[Horizontal Pod Autoscaler - 官方文档](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/)



安装：

```
# 镜像修改
# 测试环境有个证书问题，需要添加启动参数 - --kubelet-insecure-tls
# 要求，目标 工作负载，需要有 requests，才能有相关的资源采集
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

注意：

- pod中每个容器都必须制定资源限制



#### 二、CronHPA

[kubernetes-cronhpa-controller - github](https://github.com/AliyunContainerService/kubernetes-cronhpa-controller)

定时任务，到时间将replica调整到固定大小；问题：和官方HPA会冲突

[另一个项目别人提的ISSUE，问题+建议](https://github.com/tkestack/cron-hpa/issues/1)



### 滚动发布/渐进式发布/分批发布

1、Argo Rollout: 自定义工作负载，内置 Rollout能力

2、Flagger: 支持内置工作负载，升级期间资源 double

3、Kruise Rollout: 利用 pause控制



[Kruise Rollout v0.2.0 版本发布：支持 Gateway API、StatefulSet 分批发布等能力](https://mp.weixin.qq.com/s/zTqfOO9o1_c1JTtYkp82EA)

[Kruise Rollout：灵活可插拔的渐进式发布框架](https://mp.weixin.qq.com/s/P8gYuDHM7ZDWM3Y7fYSK8g)

[Argo Rollouts](https://zhuanlan.zhihu.com/p/475782364)

[argocd蓝绿/金丝雀发布之rollout](https://www.jianshu.com/p/bd185d76535f)

[Kruise Rollout - bilibili](https://www.bilibili.com/video/BV1wT4y1Q7eD)

---



维护应用负责人、测试负责人、架构师等人员信息，支持流程审批人员自动获取

