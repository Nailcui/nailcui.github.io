- Golang的runtime学习
    - 调度模型G-P-M
    - 内存分配
    - GC
    - 操作系统api、cpu操作、CGO的支持
    - pprof、trace、race检测的支持
    - map、channel、string等内置类型及反射的实现
- Golang的重点分析
    - channel
        - [Golang channel 源码深度剖析](https://www.cyhone.com/articles/analysis-of-golang-channel/)
    - defer
    - sync.Pool
        - [深度分析 Golang Sync.Pool 底层原理](https://www.cyhone.com/articles/think-in-sync-pool/)
        - 一个优化思路: [com/golang/go/issues/27735](https://github.com/golang/go/issues/27735#issuecomment-739169121)
- Golang的并发编程
    - time/rate
        - [Golang 标准库限流器 time/rate 实现剖析](https://www.cyhone.com/articles/analisys-of-golang-rate/)
    - sync.Cond
        - [Golang sync.Cond 条件变量源码分析](https://www.cyhone.com/articles/golang-sync-cond/)
    - sync.Pool
    - sync.WaitGroup
        - [Golang WaitGroup 原理深度剖析](https://www.cyhone.com/articles/golang-waitgroup/)
    - sema
    - sync.Lock
    - 定时器
        - [Golang 定时器底层实现深度剖析](https://www.cyhone.com/articles/analysis-of-golang-timer/)



### 运行、打包相关操作

指定Go协程最多可以在多少个线程上执行

```
GOMAXPROCS=2 go run main.go
```

注意，如果代码中也设置了`runtime.GOMAXPROCS(n)`，则最后是代码里的为准

