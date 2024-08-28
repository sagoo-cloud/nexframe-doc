# 随机数


## 1. 简介

本工具包提供了一系列用于生成随机数据的函数，包括随机整数、浮点数、字符串等。所有函数都使用密码学安全的随机数生成器，适合用于需要高度随机性的场景。

## 2. 安装

确保你的 Go 版本至少为 1.16。然后，你可以通过以下命令安装本包：

```
go get github.com/sagoo-cloud/nexframe
```

在你的代码中导入：

```go
import "github.com/sagoo-cloud/nexframe/utils"
```

## 3. 函数详解

### 3.1 Rand - 生成随机整数

```go
func Rand(min, max int) (int, error)
```

生成一个在 [min, max] 范围内的随机整数。

示例：
```go
num, err := utils.Rand(1, 100)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机数：%d\n", num)
```

### 3.2 RandFloat - 生成随机浮点数

```go
func RandFloat(min, max float64) (float64, error)
```

生成一个在 [min, max) 范围内的随机浮点数。

示例：
```go
num, err := utils.RandFloat(0, 1)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机浮点数：%.4f\n", num)
```

### 3.3 RandString - 生成随机字符串

```go
func RandString(length int) (string, error)
```

生成指定长度的随机字符串，包含大小写字母和数字。

示例：
```go
str, err := utils.RandString(10)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机字符串：%s\n", str)
```

### 3.4 RandStringNoNumber - 生成不包含数字的随机字符串

```go
func RandStringNoNumber(length int) (string, error)
```

生成指定长度的随机字符串，只包含大小写字母。

示例：
```go
str, err := utils.RandStringNoNumber(8)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("不含数字的随机字符串：%s\n", str)
```

### 3.5 RandLowerString - 生成小写字母随机字符串

```go
func RandLowerString(length int) (string, error)
```

生成指定长度的随机小写字母字符串。

示例：
```go
str, err := utils.RandLowerString(6)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机小写字母字符串：%s\n", str)
```

### 3.6 RandUpperString - 生成大写字母随机字符串

```go
func RandUpperString(length int) (string, error)
```

生成指定长度的随机大写字母字符串。

示例：
```go
str, err := utils.RandUpperString(6)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机大写字母字符串：%s\n", str)
```

### 3.7 RandNumber - 生成随机数字字符串

```go
func RandNumber(length int) (string, error)
```

生成指定长度的随机数字字符串。

示例：
```go
str, err := utils.RandNumber(4)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("随机数字字符串：%s\n", str)
```

### 3.8 RandIntSlice - 随机打乱整数切片

```go
func RandIntSlice(slice []int) error
```

随机打乱给定的整数切片。

示例：
```go
slice := []int{1, 2, 3, 4, 5}
err := utils.RandIntSlice(slice)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("打乱后的切片：%v\n", slice)
```

### 3.9 UniqueCode - 生成唯一码

```go
func UniqueCode(id int64, minLen int) (string, error)
```

根据给定的 ID 和最小长度生成唯一码。

示例：
```go
code, err := utils.UniqueCode(12345, 8)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("生成的唯一码：%s\n", code)
```

## 4. 注意事项

- 所有函数都是并发安全的，可以在多个 goroutine 中同时使用。
- 错误处理很重要，请始终检查返回的错误。
- `UniqueCode` 函数生成的码不保证全局唯一，但对于给定的 ID 是唯一的。

## 5. 常见问题

Q: 这些函数的随机性够强吗？
A: 是的，所有函数都使用 `crypto/rand` 包，提供密码学安全的随机性。

Q: 如何在 Web 应用中使用这些函数？
A: 这些函数非常适合用于生成会话 ID、临时密码、CSRF 令牌等。例如：

```go
func generateSessionID() string {
    id, err := utils.RandString(32)
    if err != nil {
        log.Fatal(err)
    }
    return id
}
```

Q: `UniqueCode` 函数的用途是什么？
A: `UniqueCode` 函数适用于需要根据数字 ID 生成可读性更好的标识符的场景，比如订单号、邀请码等。

