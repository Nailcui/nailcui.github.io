# 构建自己的 容器

## 一、准备工作

[namespaces(7) — Linux manual page](https://man7.org/linux/man-pages/man7/namespaces.7.html)





















## 二、开始制造自己的容器



### v0.0: 使用go，调用命令`sh` ，然后进入交互命令

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	fmt.Println("box starting...")
    
    // 准备一个命令
	cmd := exec.Command("sh")

    // 将当前的标准输入、输出作为这个命令的输入输出
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

    // 执行命令
    if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```





### v0.1: 体验一下隔离`hostname`的感觉

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	fmt.Println("box starting...")
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}

```

在linux上运行之后，会进入一个`shell` 交互命令行；在命令行中修改主机名，主机上的不会受到影响：

```sh
# 主机上执行
$ hostname
old-name

# 我们新的sh中执行
$ hostname -b new-name
$ hostname
new-name

# 再主机上验证：
$ hostname
old-name
```

可见容器内的修改不会影响到主机，UTS隔离已经生效。









```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	fmt.Println("box starting...")
	fmt.Printf("arg1: %s\n", os.Args[1])
	if "run" == os.Args[1] {
		run()
	} else if "init" == os.Args[1] {
		initContainer()
	} else {
		fmt.Printf("unknow arg: %s\n", os.Args[1])
	}
}

func run() {
	fmt.Println("box.run")
	cmd := exec.Command("/proc/self/exe", "init")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS | syscall.CLONE_NEWNET | syscall.CLONE_NEWUSER,
		UidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      syscall.Getuid(),
				Size:        1100,
			},
		},
		GidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      syscall.Getgid(),
				Size:        1100,
			},
		},
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Start(); err != nil {
		log.Fatal(err)
	}
	cmd.Wait()
	os.Exit(-1)
}

func initContainer() {
	fmt.Println("box.init")
	mountFlag := syscall.MS_NOEXEC | syscall.MS_NOSUID | syscall.MS_NODEV
	syscall.Mount("proc", "/proc", "proc", uintptr(mountFlag), "")
	// 这里必须是全路径
	if err := syscall.Exec("/bin/sh", []string{}, os.Environ()); err != nil {
		fmt.Printf("exec error: %s\n", err.Error())
	}
}
```

