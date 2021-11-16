### 示例

```

func main() {
	text := `{{$CurrentCluster := .Cluster }}cluster:{{ .Cluster }}Topic:{{ range .Topics }}-{{ $CurrentCluster }} {{ end }}`
	tmpl, err := template.New("users").Parse(text)
	if err != nil {
		panic(err)
	}
	err = tmpl.Execute(os.Stdout, struct {
		Cluster string
	}{
		Cluster: "biz",
	})
	if err != nil {
		panic(err)
	}
}

```

### 说明

#### 变量

定义变量

```
{{$CurrentCluster := .Cluster }}
```

使用变量

```
# 循环中可以用外部作用域的数据
{{ range .Topics }} - {{ $CurrentCluster }} {{ end }}
```

