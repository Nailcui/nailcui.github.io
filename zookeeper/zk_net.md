# zk网络编程模型

大概分这么几部分：

- 3.4.8 版本 zk自己的`NIOServerCnxnFactory`
- 3.6.x 版本 zk自己的`NIOServerCnxnFactory`
- NettyServerCnxnFactory



简单测试 read的读耗时性能测试结果如下，可见 `NettyServerCnxnFactory` 的性能最好

|                            | 300 并发读 |      |
| -------------------------- | ---------- | ---- |
| 3.4.8 NIOServerCnxnFactory | 120        |      |
| 3.6.x NIOServerCnxnFactory | 80         |      |
| NettyServerCnxnFactory     | 60         |      |



## 性能差异原因描述

在 3.4.x （3.5之前）的 `NIOServerCnxnFactory` ，一个线程处理 selector、网络读写、序列化、反序列化

在 3.6.x （3.5及之后）的 `NIOServerCnxnFactory` ，采用 reactor 模型



## 3.4.x `NIOServerCnxnFactory` 模型描述

```java

    public void run() {
        while (!ss.socket().isClosed()) {
            try {
                // 不论是 connect 还是 read、write 都是在这一个selector里进行触发
                selector.select(1000);
                Set<SelectionKey> selected;
                synchronized (this) {
                    selected = selector.selectedKeys();
                }
                ArrayList<SelectionKey> selectedList = new ArrayList<SelectionKey>(
                        selected);
                Collections.shuffle(selectedList);
                for (SelectionKey k : selectedList) {
                    if ((k.readyOps() & SelectionKey.OP_ACCEPT) != 0) {
                        SocketChannel sc = ((ServerSocketChannel) k.channel()).accept();
                        InetAddress ia = sc.socket().getInetAddress();
                        LOG.info("Accepted socket connection from "
                                 + sc.socket().getRemoteSocketAddress());
                        sc.configureBlocking(false);
                        SelectionKey sk = sc.register(selector, SelectionKey.OP_READ);
                        NIOServerCnxn cnxn = createConnection(sc, sk);
                        sk.attach(cnxn);
                        addCnxn(cnxn);

                    } else if ((k.readyOps() & (SelectionKey.OP_READ | SelectionKey.OP_WRITE)) != 0) {
                        NIOServerCnxn c = (NIOServerCnxn) k.attachment();
                        c.doIO(k);
                    } else {
                        if (LOG.isDebugEnabled()) {
                            LOG.debug("Unexpected ops in select " + k.readyOps());
                        }
                    }
                }
                selected.clear();
            } catch (Exception e) {
                LOG.warn("Ignoring exception", e);
            }
        }
        closeAll();
        LOG.info("NIOServerCnxn factory exited run method");
    }

```



## 3.6.x `NIOServerCnxnFactory` 模型描述



## `NettyServerCnxnFactory` 模型描述



