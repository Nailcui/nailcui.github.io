



### zk 权限控制机制

> ACL 访问控制表，比UGO控制粒度更细

zookeeper的ACL机制：权限模式（Scheme）、授权对象（id）、权限（Permission）

#### 权限模式（Scheme）

##### IP

通过ip/网段粒度的控制

##### Digest

username、password 模式的控制

##### World

##### Super

#### 授权对象（id）

#### 权限（Permission）

##### C（create）

##### D（delete）

##### R（read）

##### W（write/update）

##### A（admin）







```
新增 用户:密码
addauth digest zookeeper:admin
```



### zk admin server

