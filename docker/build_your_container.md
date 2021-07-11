

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