### 利用反射动态生成struct
```go
package my_struct

import (
	"errors"
	"reflect"
)
// 构造器
type Builder struct {
	// 用于存储属性字段
	fileId []reflect.StructField
}

func NewBuilder() *Builder {
	return &Builder{}
}
// 添加字段
func (b *Builder)AddField(field string,typ reflect.Type) *Builder {
	b.fileId = append(b.fileId,reflect.StructField{Name: field,Type: typ})
	return b
}
// 根据预先添加的字段构建出结构体
func (b *Builder)Build() *Struct {
	stu := reflect.StructOf(b.fileId)
	index := make(map[string]int)
	for i := 0; i < stu.NumField(); i++ {
		index[stu.Field(i).Name] = i
	}
	return &Struct{stu,index}
}
func (b *Builder) AddString(name string) *Builder {
	return b.AddField(name, reflect.TypeOf(""))
}

func (b *Builder) AddBool(name string) *Builder {
	return b.AddField(name, reflect.TypeOf(true))
}

func (b *Builder) AddInt64(name string) *Builder {
	return b.AddField(name, reflect.TypeOf(int64(0)))
}

func (b *Builder) AddFloat64(name string) *Builder {
	return b.AddField(name, reflect.TypeOf(float64(1.2)))
}
// 实际生成的结构体，基类
// 结构体的类型
type Struct struct {
	typ reflect.Type
	// <fieldName : 索引> // 用于通过字段名称，从Builder的[]reflect.StructField中获取reflect.StructField
	index map[string]int
}

func (s Struct)New() *Instance {
	return &Instance{reflect.New(s.typ).Elem(),s.index}
}
// 结构体的值
type Instance struct {
	instance reflect.Value
	// <fieldName : 索引>
	index map[string]int
}
var (
	FieldNoExist error = errors.New("field no exist")
)
func (in Instance)Field(name string) (reflect.Value,error) {
	if i,ok := in.index[name];ok{
		return in.instance.Field(i),nil
	}else {
		return reflect.Value{},FieldNoExist
	}
}
func (in *Instance) SetString(name, value string) {
	if i,ok := in.index[name];ok{
		in.instance.Field(i).SetString(value)
	}
}

func (in *Instance) SetBool(name string, value bool) {
	if i,ok := in.index[name];ok{
		in.instance.Field(i).SetBool(value)
	}
}

func (in *Instance) SetInt64(name string, value int64) {
	if i,ok := in.index[name];ok{
		in.instance.Field(i).SetInt(value)
	}
}

func (in *Instance) SetFloat64(name string, value float64) {
	if i,ok := in.index[name];ok{
		in.instance.Field(i).SetFloat(value)
	}
}
func (i *Instance) Interface() interface{} {
	return i.instance.Interface()
}

func (i *Instance) Addr() interface{} {
	return i.instance.Addr().Interface()
}
```

### 测试
```go
func TestStruct(t *testing.T) {
	pe := NewBuilder().
		AddString("Name").
		AddInt64("Age").
		Build()
	p := pe.New()
	p.SetString("Name","你好")
	p.SetInt64("Age",32)
	fmt.Printf("%+v\n",p)
	fmt.Printf("%T，%+v\n",p.Interface(),p.Interface())
	fmt.Printf("%T，%+v\n",p.Addr(),p.Addr())
}
```
### 结果
```go
=== RUN   TestStruct
&{instance:{typ:0xc00003e180 ptr:0xc0000045a0 flag:409} index:map[Age:1 Name:0]}
struct { Name string; Age int64 }，{Name:你好 Age:32}
*struct { Name string; Age int64 }，&{Name:你好 Age:32}
--- PASS: TestStruct (0.00s)
PASS
```
