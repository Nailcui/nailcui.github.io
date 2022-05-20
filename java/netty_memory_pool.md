### 内存分配器的主要类型：

#### 1、jemalloc

> 此分配器模型在netty、redis中有使用

#### 2、ptmalloc



#### 3、tcmalloc

> golang中使用的就是这个分配器

![image-20220321222123894](netty_memory_pool.assets/image-20220321222123894.png)

![image-20220321222205009](netty_memory_pool.assets/image-20220321222205009.png)



### `jemalloc` 和`tcmalloc` 有什么区别呢

