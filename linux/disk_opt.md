## 系统盘扩容

### 流程

1、备份快照，以备万一

2、扩容分区

3、扩容文件系统



### 操作

[阿里云教程](https://help.aliyun.com/document_detail/113316.htm)

```shell
# 查看磁盘类型，以及当前的磁盘容量
df -Th

fdisk -lu

# 1、扩容分区
# growpart device 1
growpart /dev/vda 1

# 2、扩容文件系统
# xfs 格式的磁盘: xfs_growfs mountpoint
xfs_growfs /
# ext 格式的磁盘: resize2fs device
resize2fs /dev/vda1
```

异常

```shell
# 扩容分区异常
$ growpart /dev/vda 1
unexpected output in sfdisk --version [sfdisk，来自 util-linux 2.32.1]

# 解决
$ LANG=en_US.UTF-8
```



## 挂载数据盘

[阿里云文档](https://help.aliyun.com/document_detail/25426.htm)

```shell
# 查看分区
fdisk -l

# 格式化
mkfs.ext4 /dev/vdb

# 新建挂载点
mkdir /data

# 挂载
mount /dev/vdb /data
```

