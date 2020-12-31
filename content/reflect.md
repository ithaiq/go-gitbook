# reflect反射实战
> 此开源图书由[thaiq](https://github.com/ithaiq)原创，创作不易转载请注明出处

* 定义测试数据

```go
type User struct {
	UserId   int `name:"uid"`
	UserName string
}
```

* 通过反射打印字段名和类型

```go
	u := User{}
	t := reflect.TypeOf(u)
	if t.Kind() == reflect.Ptr {
		t = t.Elem() //如果u是指针要这样用
	}
	for i := 0; i < t.NumField(); i++ {
		fmt.Println(t.Field(i).Name, t.Field(i).Type)
	}
```

* 通过反射设置字段值

```go
	u := User{}
	t := reflect.ValueOf(u)
	if t.Kind() == reflect.Ptr {
		t = t.Elem() //如果u是指针要这样用
	}
	values := []interface{}{12, "test"}
	for i := 0; i < t.NumField(); i++ {
		fmt.Println(t.Field(i).Interface())
		if t.Field(i).Kind() == reflect.ValueOf(values[i]).Kind() {
			t.Field(i).Set(reflect.ValueOf(values[i]))
		}
	}
```

* 案例（map转struct)

```go
func Map2Struct(m map[string]interface{}, u interface{}) {
	v := reflect.ValueOf(u)
	if v.Kind() == reflect.Ptr {
		v = v.Elem()
		if v.Kind() != reflect.Struct {
			panic("must struct")
		}
		findFromMap := func(key string, nameTag string) interface{} {
			for k, v := range m {
				if k == key || k == nameTag {
					return v
				}
			}
			return nil
		}
		for i := 0; i < v.NumField(); i++ {
			getTagValue := findFromMap(v.Type().Field(i).Name, v.Type().Field(i).Tag.Get("name"))
			if getTagValue != nil && reflect.ValueOf(getTagValue).Kind() == v.Field(i).Kind() {
				v.Field(i).Set(reflect.ValueOf(getTagValue))
			}
		}
	} else {
		panic("must ptr")
	}
}
```

