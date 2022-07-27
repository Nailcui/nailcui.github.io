# kubernetes gateway

将我们的服务暴露到集群外，除了service、ingress外，kubernetes又发起了一个新的项目: [`gateway-api`](https://github.com/kubernetes-sigs/gateway-api)；

这个项目的目标是通过提供表现性的、可扩展的、面向角色的接口来改善服务网络，这些接口由许多厂商实现，并得到了业界的广泛支持。

Gateway API 是一个 API 资源的集合 —— Service、GatewayClass、Gateway、HTTPRoute、TCPRoute 等。使用这些资源共同为各种网络用例建模。



