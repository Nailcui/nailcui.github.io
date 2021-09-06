## 线程池

- [Java线程池，你五分钟讲完，而我和面试官聊了半小时](https://zhuanlan.zhihu.com/p/132748927)
- [Java线程池实现原理及其在美团业务中的实践](https://zhuanlan.zhihu.com/p/123328822)



#### 线程池： 简单理解，它就是一个管理线程的池子

- 它帮我们管理线程，避免增加创建线程和销毁线程的资源损耗。因为线程其实也是一个对象，创建一个对象，需要经过类加载过程，销毁一个对象，需要走GC垃圾回收流程，都是需要资源开销的。
- 提高响应速度。 如果任务到达了，相对于从线程池拿线程，重新去创建一条线程执行，速度肯定慢很多。

- 重复利用。 线程用完，再放回池子，可以达到重复利用的效果，节省资源。
- 利用多核cpu、利用cpu更好的处理io密集任务



#### 线程池创建各个参数的含义

```java
    public ThreadPoolExecutor(
        int corePoolSize,
        int maximumPoolSize,
        long keepAliveTime,
        TimeUnit unit,
        BlockingQueue<Runnable> workQueue,
        ThreadFactory threadFactory,
        RejectedExecutionHandler handler) {}
```



- corePoolSize： 线程池核心线程数最大值
- maximumPoolSize： 线程池最大线程数大小

- keepAliveTime： 线程池中非核心线程空闲的存活时间大小
- unit： 线程空闲存活时间单位

- workQueue： 存放任务的阻塞队列
- threadFactory： 用于设置创建线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。

- handler： 线城池的饱和策略事件，主要有四种类型。



### 线程池添加一个任务

> ThreadPoolExecutor.execute

 **前提条件：** 任务 not null

**步骤**：

1、工作线程小于核心工作池，则创建核心线程池并执行；创建核心线程池时会做进一步的线程状态及线程数量校验，所以可能会添加失败；成功则返回，失败则继续向后走

2、如果能成功添加任务至队列中，仍然需要double-check：

- 线程池已关闭，则需要回滚任务，并执行拒绝策略
- 是否需要添加线程

3、1、2都不能成功则准备添加到非核心任务队列；如果失败则执行拒绝策略



**注意点**：

1、先核心线程池，然后队列，然后非核心线程池

2、核心线程池未满，新任务来临，即使当前无任务也会先创建新的核心线程



### 线程池的拒绝策略

1. ThreadPoolExecutor.AbortPolicy - 默认拒绝策略，抛出 RejectedExecutionException 异常
2. ThreadPoolExecutor.DiscardPolicy - 丢弃策略
3. ThreadPoolExecutor.DiscardOldestPolicy - 丢弃旧任务策略；移除旧任务，然后重新执行任务
4. ThreadPoolExecutor.CallerRunsPolicy - 直接执行策略；在原线程中，直接执行任务

dubbo中的拒绝策略：



参考：

- [java线程池ThreadPoolExecutor八种拒绝策略浅析](http://www.kailing.pub/article/index/arcid/255.html)
- [记线上dubbo线程池耗尽事件-"CyclicBarrier惹的祸"](http://www.kailing.pub/article/index/arcid/212.html)



#### 有哪些阻塞队列

- ArrayBlockingQueue（数组有界阻塞队列FIFO）
- LinkedBlockingQueue（链表有界阻塞队列，默认最大长度为Integer.MAX_VALUE。此队列按照先进先出的原则对元素进行排序）

- PriorityBlockingQueue（优先级队列，可以自定义排序方法，但是对于同级元素不能保证顺序
- DelayQueue（延迟获取元素队列，指定时间后获取，为无界阻塞队列。

- SynchronousQueue（不存储元素的阻塞队列。每一个put操作必须订单tabke 操作，否则不能继续添加元素。
- LinkedTransfetQueue（无界阻塞队列，多了tryTransfer 和transfet方法

- LinkedBlockingQueue（链表结构组成的双向阻塞队列。可以从队列的两端插入和移除元素



#### 常见线程池

- newWorkStealingPool

- newFixedThreadPool（固定线程大小，队列长度=Integer.MAX_VALUE）
- newSingleThreadExecutor（单个线程，队列长度=Integer.MAX_VALUE）

- newCachedThreadPool（线程核心数量0，线程最大数量Integer.MAX_VALUE）
- newSingleThreadScheduledExecutor

- newScheduledThreadPool（线程核心数量0，线程最大数量Integer.MAX_VALUE）



#### 为什么不建议使用Executors来创建线程池

让使用者更加清晰的知道自己在干什么，更清晰的去认识自己创建的线程池是什么样子的



说明：Executors返回的线程池对象的弊端如下：

1：FixedThreadPool 和 SingleThreadPool：

允许的请求队列（底层实现是LinkedBlockingQueue）长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM

2：CachedThreadPool 和 ScheduledThreadPool

允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。  



#### **execute和submit的区别**

execute 不关心返回值

submit 有返回值

