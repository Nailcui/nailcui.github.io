shell
---

## 基础语法

### 循环

```sh
# 每秒输出当前时间
while :
do
  date
  sleep 1
done
```

```shell
# 删除所有状态为 Evicted 的 pod
# 从第二行开始处理: awk 'NR>1{print $1}'
for ns in `kubectl get ns | awk 'NR>1{print $1}'`
do
    echo "deal with namespace: "$ns
    kubectl get pods -n ${ns} | grep Evicted | awk '{print $1}' | xargs kubectl -n ${ns} delete pod
done
```



## 示例

### 文件相关

```sh
#!/bin/bash

# 模糊检测文件是否存在
if [ -f /tmp/som* ]; then
	echo "exist file"
fi
echo "end"

# 检测文件是否存在
#!/bin/bash
if [ -f /tmp/somefile ]; then
	echo "exist file"
fi
echo "end"

# 检测文件夹是否存在
#!/bin/bash
if [ -d /tmp ]; then
	echo "exist dir"
fi
echo "end"

```

### 文件删除

```shell
# 删除前天及之前的文件
find /logs/app/ -name "*.log" -mtime +0 -exec rm -f {} \;

# -mtime
# +0 前天及之前
# 0 昨天
# 1 前天
# -1 前天之后
```



### 时间相关

```sh
#获取昨天日期 yyyyMMdd 格式  
yesterday=`date --date='1 days ago' +%Y%m%d`  
#获取昨天日期 yyyy-MM-dd 格式  
yesterday2=`date --date='1 days ago' +%Y-%m-%d`  

  
#获取今天日期 yyyyMMdd 格式  
today=`date --date='0 days ago' +%Y%m%d`  
#获取今天日期 yyyy-MM-dd 格式  
today2=`date --date='0 days ago' +%Y-%m-%d`  
  
  
#获取明天日期 yyyyMMdd 格式  
today=`date --date='1 days' +%Y%m%d`  
#获取明天日期 yyyy-MM-dd 格式  
today2=`date --date='1 days' +%Y-%m-%d`  
  
  
#获取当前时间 yyyy-MM-dd HH:mm:ss 格式  
nowtime=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`  
#获取当前日间 HH:mm:ss 格式  
nowtime2=`date --date='0 days ago' +%H:%M:%S`  
  
  
#获取1小时前时间 yyyy-MM-dd HH:mm:ss 格式  
onehourage=`date --date='1 hours ago' "+%Y-%m-%d %H:%M:%S"`  
#获取1小时后时间 yyyy-MM-dd HH:mm:ss 格式  
onehourage2=`date --date='1 hours' "+%Y-%m-%d %H:%M:%S"`  
  
  
#获取2小时前时间 yyyy-MM-dd HH:mm:ss 格式  
twohourage=`date --date='2 hours ago' "+%Y-%m-%d %H:%M:%S"`  
#获取2小时后时间 yyyy-MM-dd HH:mm:ss 格式  
twohourage2=`date --date='2 hours' "+%Y-%m-%d %H:%M:%S"`  
  
  
#获取1个月前时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month ago' "+%Y-%m-%d %H:%M:%S"  
#获取1个月加1天前时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month ago + 1 day ago' "+%Y-%m-%d %H:%M:%S"  
#获取1个月减1天前时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month ago - 1 day ago' "+%Y-%m-%d %H:%M:%S"  
  
  
  
#获取1个月加1天加1小时加1分钟加1秒钟前时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month ago + 1 day ago + 1 hour ago + 1 min ago + 1 sec ago' "+%Y-%m-%d %H:%M:%S"  
#获取1个月加1天减1小时减1分钟减1秒钟前时间 yyyy-MM-dd HH:mm:ss 格式  可以 + - 滥用  
date --date='1 month ago + 1 day ago - 1 hour ago - 1 min ago - 1 sec ago' "+%Y-%m-%d %H:%M:%S"  

  
#获取1个月加1天加1小时加1分钟加1秒钟后时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month + 1 day  + 1 hour  + 1 min  + 1 sec ' "+%Y-%m-%d %H:%M:%S"
  
  
#获取1个月加1天减1小时减1分钟减1秒钟后时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 month + 1 day  - 1 hour  - 1 min  - 1 sec ' "+%Y-%m-%d %H:%M:%S"
  
  
#获取1年后1个月加1天减1小时减1分钟减1秒钟后时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 year 1 month + 1 day  - 1 hour  - 1 min  - 1 sec ' "+%Y-%m-%d %H:%M:%S"
  
  
#获取1年前1个月加1天减1小时减1分钟减1秒钟后时间 yyyy-MM-dd HH:mm:ss 格式  
date --date='1 year 1 month + 1 day  - 1 hour  - 1 min  - 1 sec ' "+%Y-%m-%d %H:%M:%S"
```

### 删除确认（命令行交互）

```sh
function delete_sure(){
  cat << eof
  $(echo -e "\033[1;36m注意:\033[0m")
  Delete the file
eof

  read -p "Please reconfirm that you want to delete the file. (yes/no) " ans
  while [[ "x"$ans != "xyes" && "x"$ans != "xno" ]]; do
      read -p "Please reconfirm that you want to delete the file.  (yes/no) " ans
  done

  if [[ "x"$ans == "xno" ]]; then
      exit
  fi
  echo "delete success"
}
```

### 参数校验、判断

```sh
# 判断参数不为空
if [ ! -n "$1" ] ;then
    echo "请输入应用名"
    exit
fi


# 判断变量不是单个字符数量
if [ `echo $PID | wc -w` -ne 1 ] ;then
    echo "！！匹配进程: $PID"
    exit
fi

# 判断文件、目录是否存在
if [ -d /tmp/abc -o -f /tmp/abc ] ;then
    echo "文件/文件夹已存在"
    exit
fi


```
