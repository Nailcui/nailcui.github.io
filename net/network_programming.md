### 一、线程模型

提起网络编程，应用层最重要的就是线程模型了，其对性能起决定作用。



#### 1、one loop per thread：每个请求一个线程

此模式下，每个IO线程有一个 event loop（或者叫 Reactor）；在线程数基本固定的情况下是比较好的处理办法；



#### 2、线程池+阻塞IO



#### 3、non-blocking IO + IO multiplexing

- event loop（IO loop）用作IO多路复用，配合非阻塞io和定时器
- thread pool用来做计算，



#### 4、Leader/Follower模式



### C10k问题



### C10m问题



