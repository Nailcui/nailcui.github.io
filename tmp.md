TCP_NODELAY、Nagle、TCP_CORK，

边缘触发、水平触发



### 实现思路

#### API

- LKeys(bucket, pattern string) ([]string, err)
- SKeys(bucket, pattern string) ([]string, err)
- ZKeys(bucket, pattern string) ([]string, err)



#### desc

This series of APIs is used to find all keys matching a given pattern, similar to Redis's command: [KEYS](https://redis.io/commands/keys/)

Supported glob-style patterns:

- `*` matches all keys
- `h?llo` matches `hello`, `hallo` and `hxllo`
- `h*llo` matches `hllo` and `heeeello`
- `h[ae]llo` matches `hello` and `hallo,` but not `hillo`
- `h[^e]llo` matches `hallo`, `hbllo`, ... but not `hello`
- `h[a-b]llo` matches `hallo` and `hbllo`





## 2. 语法举例
| JSONPath	| 语义 |
|--------------------------------|-----------------------------------------------------------|
| $	| 根对象 |
| $[-1]	 | 最后元素 |
| $[:-2]	 | 第1个至倒数第2个 | 
| $[1:]	 | 第2个之后所有元素 | 
| $[1,2,3]	 | 集合中1,2,3个元素 |
