参考：

- [一口气说出9种分布式ID生成方式，面试官有点懵了 - 老刘的文章 - 知乎 ](https://zhuanlan.zhihu.com/p/107939861)

- [一线大厂的分布式唯一 ID 生成方案是什么样的 - 芋道源码的文章 - 知乎](https://zhuanlan.zhihu.com/p/140078865)



### 非中心化

- UUID
- 雪花算法
    - workerId生成
        - zk抢占临时节点
        - mysql等持久化主机&id
        - 手动指定
    - 时间回拨

### 中心化

- redis自增

- 单独id生成服务

    

UUID 数据无规律；其他的方式生成的id趋势递增