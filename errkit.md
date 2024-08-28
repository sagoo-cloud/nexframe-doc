# errkit 包使用指南

errkit 包是一个全面的 Go 错误处理解决方案，提供了丰富的功能来创建、包装、注解和处理错误。本指南将介绍如何使用 errkit 包的各个功能。

## 1. 安装

首先，您需要在项目的代码中导入它：

```go
import "github.com/sagoo-cloud/nexframe/errkit"
```

## 2. 创建错误

### 2.1 使用 New 函数

要创建一个新的错误，使用 `New` 函数：

```go
err := errkit.New("发生了一个错误")
```

这将创建一个包含错误消息、唯一错误代码和堆栈跟踪的 `Error` 对象。

### 2.2 使用预定义错误

errkit 包提供了一些预定义的错误类型：

```go
err := errkit.ErrNotFound
// 或
err := errkit.ErrPermission
// 或
err := errkit.ErrTimeout
```

## 3. 包装错误

使用 `Wrap` 函数可以在保留原始错误的同时添加额外的上下文：

```go
originalErr := someFunction()
wrappedErr := errkit.Wrap(originalErr, "执行操作时")
```

## 4. 错误注解

您可以为错误添加键值对形式的注解：

```go
err := errkit.New("数据库错误")
annotatedErr := errkit.Annotate(err, "user_id", 12345)
```

要获取注解信息：

```go
if userId, ok := errkit.GetAnnotation(annotatedErr, "user_id"); ok {
    fmt.Printf("错误发生时的用户ID：%v\n", userId)
}
```

## 5. 错误比较

errkit 包与标准库的 `errors.Is` 和 `errors.As` 函数兼容：

```go
if errors.Is(err, errkit.ErrNotFound) {
    fmt.Println("资源未找到")
}

var customErr *errkit.Error
if errors.As(err, &customErr) {
    fmt.Printf("错误代码：%s\n", customErr.Code)
}
```

## 6. 错误组

当需要处理多个可能的错误时，可以使用 `ErrorGroup`：

```go
eg := errkit.NewErrorGroup()
eg.Add(doTask1())
eg.Add(doTask2())
if err := eg.Err(); err != nil {
    fmt.Printf("任务执行失败：%v\n", err)
}
```

## 7. 异常处理

### 7.1 断言

使用 `Assert` 函数进行断言：

```go
errkit.Assert(user != nil, "用户不能为空")
```

### 7.2 恢复 panic

使用 `Recover` 函数从 panic 中恢复：

```go
func someFunction() (err error) {
    defer errkit.Recover(&err)
    // 可能会 panic 的代码
    return nil
}
```

### 7.3 模拟 try-catch

使用 `TryCatch` 函数模拟 try-catch 行为：

```go
errkit.TryCatch(
    func() error {
        // 尝试执行的代码
        return someRiskyFunction()
    },
    func(err error) {
        // 错误处理代码
        fmt.Printf("捕获到错误：%v\n", err)
    },
)
```

## 8. 服务级错误处理

### 8.1 格式化错误响应

使用 `FormatErrorResponse` 函数格式化错误以供 API 响应使用：

```go
if err != nil {
    response := errkit.FormatErrorResponse(err)
    // 将 response 作为 JSON 返回给客户端
}
```

### 8.2 错误日志记录

使用 `LogError` 函数记录详细的错误信息：

```go
if err != nil {
    errkit.LogError(err)
}
```

## 9. 最佳实践

1. 在函数返回错误时，优先使用 `Wrap` 来添加上下文信息。
2. 使用 `Annotate` 为错误添加结构化的元数据。
3. 在处理多个可能的错误源时，使用 `ErrorGroup`。
4. 在 API 层使用 `FormatErrorResponse` 来提供一致的错误响应格式。
5. 使用 `LogError` 在服务器端记录详细的错误信息，包括堆栈跟踪和上下文。

通过使用 errkit 包，您可以实现一致、强大且信息丰富的错误处理机制，提高代码的可读性和可维护性。