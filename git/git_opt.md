### 日常操作

#### tag相关

```
# 查看现有tag
git tag

# 打标签
git tag v0.1

# 打标签（打在之前的 commit 上）
git tag v0.1 2833b3

# 推送标签到远程
git push origin v0.1

# 推送所有标签到远程
git push origin --tags
```



### 后悔药

#### 丢弃工作区的修改

```shell
git checkout -- .
```



#### 放弃merge

> 如果merge时候发现冲突太多，或者操作了错误的分支；在merge未完成的时候，想要放弃本次merge的话

```shell
git merge --abort
```



#### 只移除git管理的，但不删除文件

```
# 移除单个文件
git rm --cached Dockerfile

# 移除目录
git rm --cached -r .idea
```



### 其他操作

#### 合并提交

如果当前有如下提交记录

```
00000003  修改log
00000002  修改log
00000001  init
```

最后2次都是修改了log，想让这2个合并到一个commit中

执行如下命令进行合并:

```
git rebase -i 00000001
```

将会进入一个vi编辑界面

1、将第二行及之后的行第一个单词改为s，s的含义可以看边记界面的提示；然后保存退出

2、自动进入一个编辑提交信息的vi界面，里面有我们多次的提交信息，#号开头的是会被忽略的，所以我们编辑只留下我们想要的提交信息就行



#### 对于git的submodule

方式一：

```
git clone --recurse-submodules https://github.com/apache/skywalking.git
cd skywalking/
```

方式二：

```
git clone https://github.com/apache/skywalking.git
cd skywalking/
git submodule init
git submodule update
```

