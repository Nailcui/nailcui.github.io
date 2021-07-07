go 反射
---


### 通过反射实例化对象

```go
func new(t reflect.Type) {
    reflect.New(t).Interface()
}
```

##### 示例: gorm通用分页查询

```go

func main() {
    db := models.DB.
        Model(&models.Application{}).
		Order("-id")
    query(db)
}

func query(db *gorm.DB, page *web.Page) {
    // result 根据db.Value 实例化用来接收结果
    result := reflect.New(reflect.SliceOf(reflect.TypeOf(db.Value))).Interface()

    db.Limit(page.PageSize).
		Offset((page.PageNo - 1) * page.PageSize).
		Find(result)
    db.Count(&page.Total)

    page.List = result
    page.Total = count
}
```
