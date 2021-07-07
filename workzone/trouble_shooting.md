## 排查命令

### 网络相关
netstat -nat | awk '{print $6}' | sort | uniq -c | sort -rn

## java 应用排查工具

### java

#### jsp
#### jinfo
#### jstat
#### jstack
#### jmap
#### btrace





