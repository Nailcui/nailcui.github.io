消息广播

zab的消息广播机制和标准的二阶段提交有些不一样，移除了中断事务的逻辑

标准的二进制提交在



1、选举

2、选举后-follower

- 连接leader
- 处理epochId等
- 和leader同步
- while isRunning
    - readPacket
    - processPacket

2、选举后-leader



[详细介绍处理客户端请求](https://www.cnblogs.com/aoshicangqiong/p/8029792.html)

[Zookeeper事务日志和数据：讲解日志、快照的基本逻辑](https://zhuanlan.zhihu.com/p/141522460)



Follower的处理逻辑链

**FollowerRequestProcessor -> CommitProcessor -> FinalRequestProcessor**



FollowerRequestProcessor 的处理入口

```
// zks.getFollower().request(request);
// 将数据写给Leader就完了
public class Learner {       
    void request(Request request) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream oa = new DataOutputStream(baos);
        oa.writeLong(request.sessionId);
        oa.writeInt(request.cxid);
        oa.writeInt(request.type);
        if (request.request != null) {
            request.request.rewind();
            int len = request.request.remaining();
            byte b[] = new byte[len];
            request.request.get(b);
            request.request.rewind();
            oa.write(b);
        }
        oa.close();
        QuorumPacket qp = new QuorumPacket(Leader.REQUEST, -1, baos
                .toByteArray(), request.authInfo);
        writePacket(qp, true);
    }
}
```

Follower处理后，的response是如何返回的



Leader的处理逻辑链路

**PreRequestProcessor -> SyncProcessor -> FinalRequestProcessor**





### Session

- 一个重要的数据结构sessionsWithTimeout存放sessionid和timeout的映射
- 另一个重要的数据结构sessionsById存放sessionid和SessionImpl实例的映射