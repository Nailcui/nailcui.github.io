### 保留现场

```
jmap -F -dump:format=b,file=atw.bin `jid` 
```

把整个堆dump到本地，dump失败，JVM已经僵死。

```
jmap -histo  `jid` > histo.log
```

保留histo内存快照成功;)

```
jstack `jid` > stack.log
```

JVM线程信息保存成功:)

现场保存完毕，重启应用。

