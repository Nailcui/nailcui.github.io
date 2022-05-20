[Zookeeper集群运维“避坑”指南-京东云](https://zhuanlan.zhihu.com/p/48292507)



initLimit：Follower和Leader初始化连接时，最多能进行多少轮心跳，超过就强制断开了；

- 如果快照太大，一直初始化不好，就会被强制断开，无法加入集群

syncLimit：Follower和Leader通信，请求和相应能容忍的最大间隔

tickTime：心跳毫秒间隔



影响集群性能的因素：

- 快照体积，避免超过1G，避免当作文件系统来使用
- 节点数量，避免超过10w
- 混布相互影响
- 节点问题及时处理



指标：

- 快照大小：zk_approximate_data_size
- 节点数量：zk_znode_count
- 节点连接数：zk_num_alive_connections
- 节点流量：zk_packets_received/zk_packert_sent
- zk-follower节点数量：zk_followers、zk_synced_followers以及zk_server_state



### 运维

```
# dataDir and dataLogDir 保留数量
autopurge.snapRetainCount=3

# 清理间隔（小时）
autopurge.purgeInterval=1

```



默认不清理，保留3：

```
2022-01-07 01:03:49,661 [myid:1] - INFO  [main:DatadirCleanupManager@78] - autopurge.snapRetainCount set to 3
2022-01-07 01:03:49,661 [myid:1] - INFO  [main:DatadirCleanupManager@79] - autopurge.purgeInterval set to 0
2022-01-07 01:03:49,661 [myid:1] - INFO  [main:DatadirCleanupManager@101] - Purge task is not scheduled.
```



#### 查看事务日志

```
$ bin/zkTxnLogToolkit.sh
usage: TxnLogToolkit [-dhrv] txn_log_file_name
-d,--dump      Dump mode. Dump all entries of the log file. (this is the default)
-h,--help      Print help message
-r,--recover   Recovery mode. Re-calculate CRC for broken entries.
-v,--verbose   Be verbose in recovery mode: print all entries, not just fixed ones.
-y,--yes       Non-interactive mode: repair all CRC errors without asking
```



全量查看：

```
sh /usr/local/zookeeper-3.6.3/bin/zkTxnLogToolkit.sh /tmp/zookeeper/logs/version-2/log.1
```



#### 查看快照

查看所有node，不看data

```
./zkSnapShotToolkit.sh /data/zkdata/version-2/snapshot.fa01000186d
```

查看所有node，包含data

```
./zkSnapShotToolkit.sh -d /data/zkdata/version-2/snapshot.fa01000186d
```

以json格式查看所有node

```
./zkSnapShotToolkit.sh -json /data/zkdata/version-2/snapshot.fa01000186d
```

