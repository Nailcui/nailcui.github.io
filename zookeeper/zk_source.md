0、提供了那些功能，分别用什么原理实现的，如果要你自己来实现，你要怎么做呢？包括代码和设计层面

1、启动后，分别开启了那几个线程，各自的任务是什么？

2、我对通信比较感兴趣，先来看看他的网络相关的实现







# 启动

server端的启动类有2个：

- ZooKeeperServerMain：单机模式
- QuorumPeerMain：集群模式



看下集群模式的：

```java
public class QuorumPeerMain {
    public static void main(String[] args) {
        QuorumPeerMain main = new QuorumPeerMain();
        main.initializeAndRun(args);
        LOG.info("Exiting normally");
        System.exit(0);
    }


    protected void initializeAndRun(String[] args) {
        QuorumPeerConfig config = new QuorumPeerConfig();
        if (args.length == 1) {
            config.parse(args[0]);
        }

        // 磁盘清理相关的任务
        // Start and schedule the the purge task
        DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config
                .getDataDir(), config.getDataLogDir(), config
                .getSnapRetainCount(), config.getPurgeInterval());
        purgeMgr.start();

        if (args.length == 1 && config.isDistributed()) {
            // 集群模式启动
            runFromConfig(config);
        } else {
            LOG.warn("Either no config or no quorum defined in config, running "
                    + " in standalone mode");
            // there is only server in the quorum -- run as standalone
            // 单机模式启动
            ZooKeeperServerMain.main(args);
        }
    }


    public void runFromConfig(QuorumPeerConfig config) {
       LOG.info("Starting quorum peer");
      try {
          // Server端的连接管理，和client的交互的主要功能实现
          ServerCnxnFactory cnxnFactory = null;

          if (config.getClientPortAddress() != null) {
              cnxnFactory = ServerCnxnFactory.createFactory();
              cnxnFactory.configure(config.getClientPortAddress(),
                      config.getMaxClientCnxns(),
                      false);
          }

          quorumPeer = getQuorumPeer();
          // ... 各种配置
          quorumPeer.initConfigInZKDatabase();
          quorumPeer.setCnxnFactory(cnxnFactory);
          quorumPeer.setLearnerType(config.getPeerType());
          quorumPeer.setSyncEnabled(config.getSyncEnabled());
          quorumPeer.setQuorumListenOnAllIPs(config.getQuorumListenOnAllIPs());
          quorumPeer.setQuorumCnxnThreadsSize(config.quorumCnxnThreadsSize);
          quorumPeer.initialize();
          
          // 开始启动
          quorumPeer.start();
          quorumPeer.join();
      } catch (InterruptedException e) {
          // warn, but generally this is ok
          LOG.warn("Quorum Peer interrupted", e);
      }
    }
}
public class QuorumPeer extends ZooKeeperThread implements QuorumStats.Provider {

    // 内存数据库
    private ZKDatabase zkDb;

    // 这里正式开始启动步骤
    @Override
    public synchronized void start() {
        // 从文件读取数据
        loadDataBase();
        
        // 开启、处理client连接
        startServerCnxnFactory();
        try {
            // ？
            adminServer.start();
        } catch (AdminServerException e) {
            LOG.warn("Problem starting AdminServer", e);
        }
        // 开始选举的准备工作
        startLeaderElection();

        // 这里开始执行线程的run
        super.start();
    }

    /**
     * 选举整个生命周期的状态控制：
     * - 判断当前状态，执行相关操作
     **/
    @Override
    public void run() {
        while (running) {
            switch (getPeerState()) {

            case LOOKING:
                reconfigFlagClear();
                if (shuttingDownLE) {
                    shuttingDownLE = false;
                    // 选举准备工作
                    startLeaderElection();
                }
                // 调用具体的选举算法，开启一次领导者选举
                setCurrentVote(makeLEStrategy().lookForLeader());
            case OBSERVING:
                try {
                    LOG.info("OBSERVING");
                    setObserver(makeObserver(logFactory));
                    observer.observeLeader();
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e );
                } finally {
                    observer.shutdown();
                    setObserver(null);  
                    updateServerState();
                }
                break;
            case FOLLOWING:
                try {
                    LOG.info("FOLLOWING");
                    setFollower(makeFollower(logFactory));
                    follower.followLeader();
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e);
                } finally {
                    follower.shutdown();
                    setFollower(null);
                    updateServerState();
                }
                break;
            case LEADING:
                LOG.info("LEADING");
                try {
                    setLeader(makeLeader(logFactory));
                    leader.lead();
                    setLeader(null);
                } catch (Exception e) {
                    LOG.warn("Unexpected exception",e);
                } finally {
                    if (leader != null) {
                        leader.shutdown("Forcing shutdown");
                        setLeader(null);
                    }
                    updateServerState();
                }
                break;
            }
            start_fle = Time.currentElapsedTime();

    }


}
```



# 选举逻辑







```java
public class ZKDatabase {

    protected DataTree dataTree;
    protected ConcurrentHashMap<Long, Integer> sessionsWithTimeouts;
    protected FileTxnSnapLog snapLog;
    protected long minCommittedLog, maxCommittedLog;
}
```

# 网络连接相关的分析 zk 3.5.x 及之后



zk的server端，处理client连接，主要有 nio实现和netty实现2种

## server端，处理client连接，nio的实现

1、reactor模式

2、代码上没有很好的处理 网络 和业务逻辑

```java
// server 端的主要实现都在这里
// 采用 reactor 模式，一个 boss，n 个 worker，基本的nio实现
public class NIOServerCnxnFactory extends ServerCnxnFactory {

    private AcceptThread acceptThread;
    private ConnectionExpirerThread expirerThread;
    private final Set<SelectorThread> selectorThreads = new HashSet<SelectorThread>();

    protected WorkerService workerPool;

    @Override
    public void start() {
        stopped = false;
        if (workerPool == null) {
            workerPool = new WorkerService("NIOWorker", numWorkerThreads, false);
        }
        
        // n 个 worker
        for(SelectorThread thread : selectorThreads) {
            if (thread.getState() == Thread.State.NEW) {
                thread.start();
            }
        }
        
        
        // 1 个 boss
        if (acceptThread.getState() == Thread.State.NEW) {
            acceptThread.start();
        }
        // 处理连接过期的线程，见：连接过期处理
        if (expirerThread.getState() == Thread.State.NEW) {
            expirerThread.start();
        }
    }

    // worker 线程
    class SelectorThread extends AbstractSelectThread {
        private final int id;
        // 暂存新连接，boss放进来的，还未注册到 selector上的连接
        private final Queue<SocketChannel> acceptedQueue;
        private final Queue<SelectionKey> updateQueue;
        
        // 当selector监听到连接有读写事件后，在这里统一处理：
        // 包装一个 Request任务，WorkerService来统一调度处理处理
        private void handleIO(SelectionKey key) {

            // 这个事件里就是调用 NIOServerCnxn.doIO 来处理网络读写，读到 ByteBuffer里，

            // 最终读取到的数据 Request 由 ZooKeeperServer.processPacket 来处理
            IOWorkRequest workRequest = new IOWorkRequest(this, key);
            NIOServerCnxn cnxn = (NIOServerCnxn) key.attachment();

            // Stop selecting this key while processing on its
            // connection
            cnxn.disableSelectable();
            key.interestOps(0);
            touchCnxn(cnxn);
            workerPool.schedule(workRequest);
        }
    }


    private class IOWorkRequest extends WorkerService.WorkRequest {
        private final SelectorThread selectorThread;
        private final SelectionKey key;
        private final NIOServerCnxn cnxn;

        public void doWork() throws InterruptedException {
            // ...
            if (key.isReadable() || key.isWritable()) {
                // NIOServerCnxn.doIO
                // 处理IO逻辑：
                //   1、如果是连接请求（第一个请求）：zkServer.processConnectRequest
                //   2、如果是正常请求：zkServer.processPacket
                cnxn.doIO(key);
                // ...
            }
            // ...
}
```



## server端，处理client连接，netty的实现

还是没有搞 decode，还用了nio的ByteBuffer；可能是为了复用老的代码逻辑





## server 间通信

```java
public class FastLeaderElection implements Election {

    LinkedBlockingQueue<ToSend> sendqueue;
    LinkedBlockingQueue<Notification> recvqueue;
}
```

# 选举



looking:

广播自己的投票

进入 loop：







```java
/**
 * 选举所用连接管理，里面是 server 间交互的代码，写的太难受，完全没有一点业务和网络分离的东西。。。
 **/
public class QuorumCnxManager {
    
    // 各个连接的发送线程
    final ConcurrentHashMap<Long, SendWorker> senderWorkerMap;
    
    // 发送数据的缓冲区，各个发送线程从这里取数据，再发送
    final ConcurrentHashMap<Long, ArrayBlockingQueue<ByteBuffer>> queueSendMap;

    // RecvWorker 收到数据后，放到这里
    public final ArrayBlockingQueue<Message> recvQueue;

}
public class FastLeaderElection implements Election {

    // 待发送的投票信息
    LinkedBlockingQueue<ToSend> sendqueue;

    // 收到的投票信息都在这里
    LinkedBlockingQueue<Notification> recvqueue;

    protected class Messenger {

        /**
         * 从QuorumCnxManager.recvQueue取数据，判断数据并做相关处理
         * 收到数据不正常、发送通知数据
         * 如果自己是Looking，把数据放到 FastLeaderElection.recvqueue
         * 如果自己不是Looking，别人是Looking，将现在的leader数据发送给对方
         **/
        class WorkerReceiver extends ZooKeeperThread  {
            QuorumCnxManager manager;
        }
        
        /**
         * 从 FastLeaderElection.sendqueue 中取数据
         * 然后通过manager将数据发送到对应的队列中，由QuorumCnxManager.SendWorker 取发送
         **/
        class WorkerSender extends ZooKeeperThread {
            QuorumCnxManager manager;
        
        }

    }

    /**
     * 默认选举算法的具体实现
     **/
	public Vote lookForLeader() throws InterruptedException {
    
    }


    // 对比，看下 newId 和自己的 curId，谁赢得这个
    protected boolean totalOrderPredicate(
        long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
        /*
         * epoch 大
         * epoch 相同，就比较 zxid
         * zxid  相同，就比较 serverId
         * We return true if one of the following three cases hold:
         * 1- New epoch is higher
         * 2- New epoch is the same as current epoch, but new zxid is higher
         * 3- New epoch is the same as current epoch, new zxid is the same
         *  as current zxid, but server id is higher.
         */

        return (
            	(newEpoch > curEpoch) ||
                (
                    (newEpoch == curEpoch) &&
                	(
                        (newZxid > curZxid) ||
                        ((newZxid == curZxid) && (newId > curId))
                    )
                )
        );
    }
}
```





成为Leader之后，执行 lead():

- loadDatabase
- 等待所有的follower提交自己的epoch，过半后推算出来新的epoch

- 设置zxid
- 等待过半的 epochAck

- 等待过半的 newLeaderAck



![img](https://cdn.nlark.com/yuque/__puml/1d4142d70fab060aa5afb62c1f977c7e.svg)





### 顺序一致性的保证

客户端连接后，校验客户端的zxid和本地的zxid，如果客户端的更大，说明本地的数据较老，拒绝连接

```java
public class NIOServerCnxn extends ServerCnxn {
    /** Read the request payload (everything following the length prefix) */
    private void readPayload() throws IOException, InterruptedException {
        if (incomingBuffer.remaining() != 0) { // have we read length bytes?
            // xxx
        }

        if (incomingBuffer.remaining() == 0) { // have we read length bytes?
            packetReceived();
            incomingBuffer.flip();
            if (!initialized) {
                // 第一次连接后，客户端的 lastZxidSeen
                readConnectRequest();
            } else {
                readRequest();
            }
            lenBuffer.clear();
            incomingBuffer = lenBuffer;
        }
    }
}
```



# 处理客户端请求

### 

```java
// 入口
public class NIOServerCnxn extends ServerCnxn {

    private void readRequest() throws IOException {
        zkServer.processPacket(this, incomingBuffer);
    }
}
public class ZooKeeperServer implements SessionExpirer, ServerStats.Provider {
    public void processPacket(ServerCnxn cnxn, ByteBuffer incomingBuffer) throws IOException {
        if (h.getType() == OpCode.auth) {
            // 认证请求
        } else if (h.getType() == OpCode.sasl) {
            // sasl
            processSasl(incomingBuffer, cnxn, h);
        } else {
            if (shouldRequireClientSaslAuth() && !hasCnxSASLAuthenticated(cnxn)) {
                // 关掉
                ReplyHeader replyHeader = new ReplyHeader(h.getXid(), 0, Code.SESSIONCLOSEDREQUIRESASLAUTH.intValue());
                cnxn.sendResponse(replyHeader, null, "response");
                cnxn.sendCloseSession();
                cnxn.disableRecv();
            } else {
                // ...
                si.setOwner(ServerCnxn.me);
                // 
                submitRequest(si);
            }
        }
    }

    public void submitRequest(Request si) {
        enqueueRequest(si);
    }

    public void enqueueRequest(Request si) {
        // 请求 节流阀, 通过节流阀后还是调用了下方的 submitRequestNow 来处理
        requestThrottler.submitRequest(si);
    }

    public void submitRequestNow(Request si) {
        setLocalSessionFlag(si);
        firstProcessor.processRequest(si);
        if (si.cnxn != null) {
            incInProcess();
        }
    }
}

```



```java
public class RequestThrottler extends ZooKeeperCriticalThread {

    private final LinkedBlockingQueue<Request> submittedRequests = new LinkedBlockingQueue<Request>();

    public void submitRequest(Request request) {
        if (stopping) {
            LOG.debug("Shutdown in progress. Request cannot be processed");
            dropRequest(request);
        } else {
            submittedRequests.add(request);
        }
    }

    @Override
    public void run() {
        try {
            while (true) {
                if (killed) {
                    break;
                }

                Request request = submittedRequests.take();
                if (Request.requestOfDeath == request) {
                    break;
                }

                if (request.mustDrop()) {
                    continue;
                }

                // Throttling is disabled when maxRequests = 0
                if (maxRequests > 0) {
                    while (!killed) {
                        if (dropStaleRequests && request.isStale()) {
                            // Note: this will close the connection
                            dropRequest(request);
                            ServerMetrics.getMetrics().STALE_REQUESTS_DROPPED.add(1);
                            request = null;
                            break;
                        }
                        if (zks.getInProcess() < maxRequests) {
                            break;
                        }
                        throttleSleep(stallTime);
                    }
                }

                if (killed) {
                    break;
                }

                // A dropped stale request will be null
                if (request != null) {
                    if (request.isStale()) {
                        ServerMetrics.getMetrics().STALE_REQUESTS.add(1);
                    }
                    // 最后还是扔回了 ZooKeeperServer去处理
                    zks.submitRequestNow(request);
                }
            }
        } catch (InterruptedException e) {
            LOG.error("Unexpected interruption", e);
        }
        int dropped = drainQueue();
        LOG.info("RequestThrottler shutdown. Dropped {} requests", dropped);
    }

}
```



## Follower逻辑

> FollowerRequestProcessor -> CommitProcessor -> FinalRequestProcessor

```java
public class FollowerRequestProcessor extends ZooKeeperCriticalThread implements RequestProcessor {

    RequestProcessor nextProcessor;

    LinkedBlockingQueue<Request> queuedRequests = new LinkedBlockingQueue<Request>();

    // 主处理逻辑
    @Override
    public void run() {
        try {
            while (!finished) {
                Request request = queuedRequests.take();
                // 发送给leader前就先去等待 Leader 返回数据
                nextProcessor.processRequest(request);
                switch (request.type) {
                case OpCode.sync:
                    zks.pendingSyncs.add(request);
                    zks.getFollower().request(request);
                    break;
                case OpCode.create:
                case OpCode.create2:
                // ....
                case OpCode.check:
                    // getFollower 就是获取当前自己这个Follower
                    // 包装为 Leader.REQUEST 数据包，发送给leader
                    zks.getFollower().request(request);
                    break;
                case OpCode.createSession:
                case OpCode.closeSession:
                    // Don't forward local sessions to the leader.
                    if (!request.isLocalSession()) {
                        zks.getFollower().request(request);
                    }
                    break;
                }
            }
        }
    }
    

    // 入口
    public void processRequest(Request request) {
        processRequest(request, true);
    }

    // 添加到队列
    void processRequest(Request request, boolean checkForUpgrade) {
        if (!finished) {
            if (checkForUpgrade) {
                // Before sending the request, check if the request requires a
                // global session and what we have is a local session. If so do
                // an upgrade.
                Request upgradeRequest = null;
                try {
                    upgradeRequest = zks.checkUpgradeSession(request);
                } catch (KeeperException ke) {
                }
            }

            queuedRequests.add(request);
        }
    }
}
```

```java
public class CommitProcessor extends ZooKeeperCriticalThread implements RequestProcessor {

    /**
     */
    protected LinkedBlockingQueue<Request> queuedRequests = new LinkedBlockingQueue<Request>();

    /**
     */
    protected final LinkedBlockingQueue<Request> queuedWriteRequests = new LinkedBlockingQueue<>();
    protected final LinkedBlockingQueue<Request> committedRequests = new LinkedBlockingQueue<Request>();

    // 如果是读请求，则CommitProcessor会继续交给FinalRequestProcessor处理，把数据读取后并返回响应包
    // 如果是写请求，则CommitProcessor会把request缓存到queuedRequests中，等待Leader发送commit请求之后再交给FinalRequestProcessor来修改本地内存状态
    @Override
    public void processRequest(Request request) {
        if (stopped) {
            return;
        }
        request.commitProcQueueStartTime = Time.currentElapsedTime();
        queuedRequests.add(request);
        if (needCommit(request)) {
            queuedWriteRequests.add(request);
            numWriteQueuedRequests.incrementAndGet();
        } else {
            numReadQueuedRequests.incrementAndGet();
        }
        wakeup();
    }

}
```



## Leader逻辑

PrepRequestProcessor -> ProposalRequestProcessor -> CommitProcessor -> ToBeAppliedRequestProcessor -> FinalRequestProcessor



#### PrepRequestProcessor

```java
public class PrepRequestProcessor extends ZooKeeperCriticalThread implements RequestProcessor {
    protected void pRequest(Request request) throws RequestProcessorException {
        request.hdr = null;
        request.txn = null;
        
        try {
            switch (request.type) {
                case OpCode.create:
                CreateRequest createRequest = new CreateRequest();
                pRequest2Txn(request.type, zks.getNextZxid(), request, createRequest, true);
                break;
                ...    
            }
        }
        // 最终交由下一个processor处理
        request.zxid = zks.getZxid();
        nextProcessor.processRequest(request);
    }
    
    // 具体处理在这里
    protected void pRequest2Txn(int type, long zxid, Request request, Record record, boolean deserialize)
        throws KeeperException, IOException, RequestProcessorException
    {
        request.hdr = new TxnHeader(request.sessionId, request.cxid, zxid,
                                    Time.currentWallTime(), type);
 
        switch (type) {
            case OpCode.create:                
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                CreateRequest createRequest = (CreateRequest)record;   
                if(deserialize)
                    // 将客户端的请求体反序列化到CreateRequest对象中
                    ByteBufferInputStream.byteBuffer2Record(request.request, createRequest);
                // path检查
                String path = createRequest.getPath();
                int lastSlash = path.lastIndexOf('/');
                if (lastSlash == -1 || path.indexOf('\0') != -1 || failCreate) {
                    LOG.info("Invalid path " + path + " with session 0x" +
                            Long.toHexString(request.sessionId));
                    throw new KeeperException.BadArgumentsException(path);
                }
                // ACL权限检查
                List<ACL> listACL = removeDuplicates(createRequest.getAcl());
                if (!fixupACL(request.authInfo, listACL)) {
                    throw new KeeperException.InvalidACLException(path);
                }
                String parentPath = path.substring(0, lastSlash);
                ChangeRecord parentRecord = getRecordForPath(parentPath);
 
                checkACL(zks, parentRecord.acl, ZooDefs.Perms.CREATE,
                        request.authInfo);
                int parentCVersion = parentRecord.stat.getCversion();
                // 根据创建节点类型，重置path信息
                CreateMode createMode =
                    CreateMode.fromFlag(createRequest.getFlags());
                if (createMode.isSequential()) {
                    path = path + String.format(Locale.ENGLISH, "%010d", parentCVersion);
                }
                validatePath(path, request.sessionId);
                try {
                    if (getRecordForPath(path) != null) {
                        throw new KeeperException.NodeExistsException(path);
                    }
                } catch (KeeperException.NoNodeException e) {
                    // ignore this one
                }
                // 检查父节点是否临时节点
                boolean ephemeralParent = parentRecord.stat.getEphemeralOwner() != 0;
                if (ephemeralParent) {
                    throw new KeeperException.NoChildrenForEphemeralsException(path);
                }
                int newCversion = parentRecord.stat.getCversion()+1;
                
                // 补充request的txn对象信息，后续requestProcessor会用到
                request.txn = new CreateTxn(path, createRequest.getData(),
                        listACL,
                        createMode.isEphemeral(), newCversion);
                StatPersisted s = new StatPersisted();
                if (createMode.isEphemeral()) {
                    s.setEphemeralOwner(request.sessionId);
                }
                // 修改父节点的stat信息
                parentRecord = parentRecord.duplicate(request.hdr.getZxid());
                parentRecord.childCount++;
                parentRecord.stat.setCversion(newCversion);
                addChangeRecord(parentRecord);
                addChangeRecord(new ChangeRecord(request.hdr.getZxid(), path, s,
                        0, listACL));
                break;
        }
        ...
        
}


```



#### ProposalRequestProcessor

```java
public class ProposalRequestProcessor implements RequestProcessor {
 
    public void processRequest(Request request) throws RequestProcessorException {
        // 如果请求来自leaner
        if(request instanceof LearnerSyncRequest){
            zks.getLeader().processSync((LearnerSyncRequest)request);
        } else {
            	// 事务和非事务请求都会将该请求流转到下一个processor（CommitProcessor ），
                nextProcessor.processRequest(request);
            // 而针对事务请求的话(事务请求头不为空)，则还需要进行事务投票等动作，在这里与之前非事务请求有所不同
            if (request.hdr != null) {
                try {
                    // 针对事务请求发起一次propose，具体在2.1
                    zks.getLeader().propose(request);
                } catch (XidRolloverException e) {
                    throw new RequestProcessorException(e.getMessage(), e);
                }
                // 将本次事务请求记录到事务日志中去
                syncProcessor.processRequest(request);
            }
        }
    }
}
```



#### 接收 Follower 回复的逻辑

leader处理 其他节点请求的代码在 `LearnerHandler` 里

```java
public class LearnerHandler extends ZooKeeperThread {
    @Override
    public void run() {
     	...
        while (true) {
            qp = new QuorumPacket();
            ia.readRecord(qp, "packet");
         	ByteBuffer bb;
            long sessionId;
            int cxid;
            int type;
 
            // 接收到响应
            switch (qp.getType()) {
                // ACK类型，说明follower已经完成该次请求事务日志的记录    
                case Leader.ACK:
                    if (this.learnerType == LearnerType.OBSERVER) {
                        if (LOG.isDebugEnabled()) {
                            LOG.debug("Received ACK from Observer  " + this.sid);
                        }
                    }
                    syncLimitCheck.updateAck(qp.getZxid());
                    // leader计算是否已经有足够的follower返回ack
                    leader.processAck(this.sid, qp.getZxid(), sock.getLocalSocketAddress());
                    break;   
                    ...
            }
        }
    }
}
```





# ZKDatabase



```java

```

