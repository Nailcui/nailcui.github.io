# golang 切片sclice

golang中的切片约等于 java中的 `ArrayList`，提供一种“动态数组“的能力。

特点：

- 非线程安全
- 自动扩容



## 切片的内部实现

```go
type slice struct {
    array unsafe.Pointer	// 指向数组的指针
    len   int				// 数据长度，数组已使用的长度
    cap   int				// 当前数组的总长度
}
```

### 切片的自动扩容

- 如果新申请容量比两倍原有容量大，那么扩容后容量大小 等于 新申请容量
- 如果原有 slice 长度小于 1024， 那么每次就扩容为原来的 2 倍
- 如果原 slice 大于等于 1024， 那么每次扩容就扩为原来的 1.25 倍



## 切片的使用

### 创建、初始化

1、直接声明的方式

```go
var names []int

// 此时长度是0，cap是0
assertions.Equal(0, len(names))
assertions.Equal(0, cap(names))
```

2、使用字面量

```go
names := []int{0, 1, 2}

// 此时长度是3，cap是3
assertions.Equal(3, len(names))
assertions.Equal(3, cap(names))
```

3、使用make

```go
// make([]T, len, cap)
names := make([]int, 3, 20)

// 此时长度是3，cap是20
assertions.Equal(3, len(names))
assertions.Equal(20, cap(names))
```

```
// make([]T, size)
names := make([]int, 3)

// 此时 len = cap = 3
assertions.Equal(3, len(names))
assertions.Equal(3, cap(names))
```



4、从现有切片/现有数组上截取，共用数组，任一修改后全部生效

```go
arr := []int{0, 1, 2, 3, 4}

names := arr[:2]
// 此时len是截取的前2个，cap=原数组的cap=5

names2 := arr[:2:3]
// 此时len是截取的前2个，cap=3
```



### 增

```go
arr := []int{0, 1, 2, 3, 4}
arr = append(arr, 2)
```



### 删

```go
func main() {
    // 初始化
	arr := []int{0, 1, 2, 3, 4}
	fmt.Printf("%p, %v\n", arr, arr)
    // 0xc000108030, [0 1 2 3 4]

    // 删除最后一个元素
	arr = arr[:len(arr)-1]
	fmt.Printf("%p, %v\n", arr, arr)
    // 0xc000108030, [0 1 2 3]

    // 删除索引为1的元素
	arr = append(arr[:1], arr[2:]...)
	fmt.Printf("%p, %v\n", arr, arr)
    // 0xc000108030, [0 2 3]
    
}
```

### 移动

```go
func main() {
	arr := []int{0, 1, 2, 3, 4}
	fmt.Printf("%p, %v\n", arr, arr)
    // 0xc0000aa060, [0 1 2 3 4]

    // 将2、3、4往前移动一位
	_ = append(arr[1:1], arr[2:]...)
	fmt.Printf("%p, %v\n", arr, arr)
    // 0xc0000aa060, [0 2 3 4 4]
}

```



### 查

```
// 通过下标访问
v := arr[1]
```



### 改

```
// 通过下标更新
arr[1] = 2
```



### 遍历

```go
// 三种遍历方式
func main() {
	arr := []int{0, 1, 2, 3, 5}
	fmt.Printf("%p, %v\n", arr, arr)

	for i := 0; i < len(arr); i++ {
	}

	for i := range arr {
	}

	for i, v := range arr {
	}
}

```



### 反转

```go
func main() {
	arr := []int{0, 1, 2, 3, 4}
	fmt.Printf("%p, %v\n", arr, arr)

	for l, r := 0, len(arr)-1; l < r; l, r = l+1, r-1 {
		arr[l], arr[r] = arr[r], arr[l]
	}
	fmt.Printf("%p, %v\n", arr, arr)
}

```





## 防止内存泄漏

当多个切片共用底层数组时，如果大切片可回收，小切片不可回收，造成底层数组无法回收，大的空间都浪费了