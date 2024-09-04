# gvar 包开发手册

## 1. 简介

gvar 包提供了一个通用的变量类型实现，称为 `Var`。这个类型可以存储任意类型的值，并提供了并发安全的操作选项。本手册将详细介绍 gvar 包的使用方法、主要功能和注意事项。

## 2. Var 结构体

### 2.1 定义

```go
type Var struct {
    value interface{} // 底层值
    safe  bool        // 是否并发安全
}
```

`Var` 结构体包含两个字段：
- `value`：存储实际的值，类型为 `interface{}`，可以存储任意类型的数据。
- `safe`：布尔值，表示是否启用并发安全模式。

### 2.2 创建新的 Var

使用 `New` 函数创建新的 `Var` 实例：

```go
func New(value interface{}, safe ...bool) *Var
```

示例：

```go
// 创建一个非并发安全的 Var
v1 := gvar.New(42)

// 创建一个并发安全的 Var
v2 := gvar.New("hello", true)
```

## 3. 主要方法

### 3.1 Set 方法

`Set` 方法用于设置 `Var` 的值，并返回旧值。

```go
func (v *Var) Set(value interface{}) (old interface{})
```

示例：

```go
v := gvar.New(10, true)
oldValue := v.Set(20)
fmt.Printf("旧值：%v，新值：%v\n", oldValue, v.Val())
```

### 3.2 Val 方法

`Val` 方法返回 `Var` 的当前值。

```go
func (v *Var) Val() interface{}
```

示例：

```go
v := gvar.New("hello")
value := v.Val()
fmt.Printf("当前值：%v\n", value)
```

### 3.3 Int 方法

`Int` 方法将 `Var` 的值转换为 int 类型。

```go
func (v *Var) Int() int
```

示例：

```go
v := gvar.New(42)
intValue := v.Int()
fmt.Printf("整数值：%d\n", intValue)
```

## 4. 并发安全

当创建 `Var` 实例时，可以指定是否启用并发安全模式。在并发安全模式下，所有的操作都会使用原子操作来确保线程安全。

示例：

```go
v := gvar.New(0, true) // 创建并发安全的 Var

var wg sync.WaitGroup
n := 1000

wg.Add(n)
for i := 0; i < n; i++ {
    go func() {
        defer wg.Done()
        current := v.Int()
        v.Set(current + 1)
    }()
}
wg.Wait()

fmt.Printf("最终结果：%d\n", v.Int())
```

## 5. 类型转换

gvar 包提供了多种类型转换方法，使得 `Var` 可以方便地转换为多种基本类型。

### 5.1 基本类型转换

- `Bool()`：转换为 bool 类型
- `Int()`：转换为 int 类型
- `Int8()`：转换为 int8 类型
- `Int16()`：转换为 int16 类型
- `Int32()`：转换为 int32 类型
- `Int64()`：转换为 int64 类型
- `Uint()`：转换为 uint 类型
- `Uint8()`：转换为 uint8 类型
- `Uint16()`：转换为 uint16 类型
- `Uint32()`：转换为 uint32 类型
- `Uint64()`：转换为 uint64 类型
- `Float32()`：转换为 float32 类型
- `Float64()`：转换为 float64 类型

示例：

```go
v := gvar.New("42")
fmt.Printf("Int: %d\n", v.Int())
fmt.Printf("Float64: %f\n", v.Float64())
fmt.Printf("Bool: %v\n", v.Bool())
```

### 5.2 时间相关转换

- `Time()`：转换为 time.Time 类型
- `Duration()`：转换为 time.Duration 类型

示例：

```go
v1 := gvar.New("2023-05-01 15:30:00")
fmt.Printf("Time: %v\n", v1.Time())

v2 := gvar.New("1h30m")
fmt.Printf("Duration: %v\n", v2.Duration())
```

## 6. JSON 支持

`Var` 类型实现了 `json.Marshaler` 和 `json.Unmarshaler` 接口，可以方便地进行 JSON 序列化和反序列化。

示例：

```go
v := gvar.New(map[string]interface{}{"name": "Alice", "age": 30})

// 序列化
jsonData, _ := json.Marshal(v)
fmt.Printf("JSON: %s\n", jsonData)

// 反序列化
var newVar gvar.Var
json.Unmarshal(jsonData, &newVar)
fmt.Printf("反序列化后的值: %v\n", newVar.Val())
```

## 7. 深拷贝

`Var` 类型支持深拷贝操作，可以创建值的完整副本。

```go
func (v *Var) DeepCopy() interface{}
```

示例：

```go
original := gvar.New([]int{1, 2, 3})
copied := original.DeepCopy().(*gvar.Var)

// 修改原始值
originalSlice := original.Val().([]int)
originalSlice[0] = 100

fmt.Printf("Original: %v\n", original.Val())
fmt.Printf("Copied: %v\n", copied.Val())
```

## 8. 注意事项

1. 在并发环境中，务必使用并发安全模式创建 `Var` 实例。
2. 类型转换方法（如 `Int()`, `Float64()` 等）在无法转换时会返回零值，使用时需要注意。
3. 使用 `DeepCopy()` 方法时，需要根据实际存储的值类型进行相应的类型断言。
4. 在性能敏感的场景中，可以考虑使用非并发安全模式，但需要自行确保线程安全。
5. 由于移除了 Add 方法，在需要进行累加操作时，需要使用 Set 方法配合当前值来实现，这在高并发情况下可能不够准确。

## 9. 结语

gvar 包提供了一个灵活、强大的通用变量类型实现。通过本手册的介绍，您应该能够在项目中有效地使用 gvar 包。如果遇到任何问题或需要进一步的帮助，请随时查阅源代码或寻求支持。