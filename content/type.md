# type实战
> 此开源图书由[thaiq](https://github.com/ithaiq)原创，创作不易转载请注明出处

* 大结构体初始化

```go
type User struct {
	Id, Sex int
	Name    string
}
type UserAttrFunc func(user *User)
type UserAttrFuncs []UserAttrFunc

func (this UserAttrFuncs) Apply(u *User) {
	for _, f := range this {
		f(u)
	}
}
func NewUser(fs ...UserAttrFunc) *User {
	u := new(User)
	//for _, f := range fs {
	//	f.Apply(u)
	//}
	UserAttrFuncs(fs).Apply(u)
	return u
}

func WithUserID(id int) UserAttrFunc {
	return func(u *User) {
		u.Id = id
	}
}

func WithUserName(name string) UserAttrFunc {
	return func(u *User) {
		u.Name = name
	}
}

func main() {
	NewUser(
		WithUserID(1),
		WithUserName("test"),
	)
}

```