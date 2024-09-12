# 微服务的序列化

## 服务同目录下序列化

可序列化类型
当您调用组件的方法时，传递给方法的参数（以及方法返回的结果）可能会被序列化并通过网络发送。因此，组件的方法只能接收和服务编织器（Service Weaver）知道如何序列化的类型，我们称之为可序列化类型。如果组件方法接收或返回的类型不是可序列化的，编织器在代码生成时会产生错误。以下是可序列化的类型：

所有原始类型（例如，int、bool、string）都是可序列化的。
如果t是可序列化的，则指针类型*t也是可序列化的。
如果t是可序列化的，则数组类型[N]t也是可序列化的。
如果t是可序列化的，则切片类型[]t也是可序列化的。
如果k和v都是可序列化的，则映射类型map[k]v也是可序列化的。
如果t不是递归的，并且满足以下一个或多个条件，则命名类型t在类型t u中是可序列化的：
t是一个协议缓冲区（即*t实现了proto.Message）；
t实现了encoding.BinaryMarshaler和encoding.BinaryUnmarshaler；
u是可序列化的；或者
u是一个嵌入了weaver.AutoMarshal的结构体类型（见下文）。
以下类型不是可序列化的：

通道类型chan t不是可序列化的。
结构字面量类型struct{...}不是可序列化的。
函数类型func(...)不是可序列化的。
接口类型interface{...}不是可序列化的。
注意：默认情况下，不实现proto.Message或BinaryMarshaler和BinaryUnmarshaler的命名结构体类型不是可序列化的。然而，它们可以通过嵌入weaver.AutoMarshal轻松地变得可序列化。

```go
type Pair struct {
    weaver.AutoMarshal
    x, y int
}
```
嵌入weaver.AutoMarshal指示编织器生成为该结构体生成序列化方法。然而，请注意，weaver.AutoMarshal并不能神奇地使任何类型变得可序列化。例如，以下代码因为NotSerializable结构体根本不可序列化，编织器会报错。

```go
// 错误：NotSerializable不能变得可序列化。
type NotSerializable struct {
    weaver.AutoMarshal
    f func()   // 函数不是可序列化的
    c chan int // chans不是可序列化的
}
```
还要注意，weaver.AutoMarshal不能嵌入到泛型结构体中。

```go
// 错误：不能在泛型结构体中嵌入weaver.AutoMarshal。
type Pair[A any] struct {
    weaver.AutoMarshal
    x A
    y A
}
```
要序列化泛型结构体，实现BinaryMarshaler和BinaryUnmarshaler。

## 不同目录下的序列化

model与weaver的服务不在同一个目录下，需要实现自定义的序列化接口，如下面的示例：

```go
type User struct {
    weaver.AutoMarshal
    Name string `json:"name"`
    Age  int    `json:"age"`
}
func (User) WeaverMarshal(*codegen.Encoder)   {}
func (User) WeaverUnmarshal(*codegen.Decoder) {}

```