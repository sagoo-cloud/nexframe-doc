# Signals 基于pub-sub 类型的进程内事件系统

## 1. 简介

Signals 包提供了一个灵活的信号系统，允许在 Go 程序中实现事件驱动的编程模式。该包支持同步和异步的信号发射，并提供了强大的错误处理和上下文取消功能。

## 2. 主要特性

- 支持同步和异步信号
- 泛型实现，可用于任何类型的数据
- 强大的错误处理，包括 panic 恢复
- 支持上下文取消
- 线程安全

## 3. 主要接口和类型

### 3.1 Signal 接口

`Signal` 是包的核心接口，定义了信号的基本操作。

```go
type Signal[T any] interface {
    Emit(ctx context.context, payload T) error
    AddListener(handler SignalListener[T], key ...string) int
    RemoveListener(key string) int
    Reset()
    Len() int
    IsEmpty() bool
}
```

### 3.2 SignalListener

`SignalListener` 是监听器函数的类型定义。

```go
type SignalListener[T any] func(context.Context, T)
```

## 4. 使用方法

### 4.1 创建信号

#### 同步信号

```go
syncSignal := signals.NewSync[string]()
```

#### 异步信号

```go
asyncSignal := signals.New[int]()
```

### 4.2 添加监听器

```go
syncSignal.AddListener(func(ctx context.Context, payload string) {
    fmt.Println("Received:", payload)
})

asyncSignal.AddListener(func(ctx context.Context, payload int) {
    fmt.Println("Received:", payload)
}, "myListener")
```

### 4.3 发射信号

```go
ctx := context.Background()

err := syncSignal.Emit(ctx, "Hello, World!")
if err != nil {
    fmt.Println("Error:", err)
}

err = asyncSignal.Emit(ctx, 42)
if err != nil {
    fmt.Println("Error:", err)
}
```

### 4.4 移除监听器

```go
asyncSignal.RemoveListener("myListener")
```

### 4.5 重置信号

```go
syncSignal.Reset()
```

## 5. 高级用法

### 5.1 使用上下文控制

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

err := asyncSignal.Emit(ctx, 100)
if err != nil {
    if err == context.DeadlineExceeded {
        fmt.Println("信号发射超时")
    } else {
        fmt.Println("发生错误:", err)
    }
}
```

### 5.2 处理 panic

Signals 包会自动捕获并处理监听器中的 panic，将其转换为错误返回。

```go
signal := signals.New[int]()

signal.AddListener(func(ctx context.Context, payload int) {
    if payload == 0 {
        panic("不能处理 0")
    }
    fmt.Println(100 / payload)
})

err := signal.Emit(context.Background(), 0)
if err != nil {
    fmt.Println("捕获到错误:", err) // 将输出: 捕获到错误: listener panicked
}
```

## 6. 完整示例

以下是一个结合了多个特性的完整示例：

```go
package main

import (
    "context"
    "fmt"
    "time"
    "github.com/sagoo-cloud/nexframe/signals"
)

func main() {
    // 创建异步信号
    numberSignal := signals.New[int]()

    // 添加监听器
    numberSignal.AddListener(func(ctx context.Context, n int) {
        fmt.Printf("监听器 1 收到: %d\n", n)
    })

    numberSignal.AddListener(func(ctx context.Context, n int) {
        fmt.Printf("监听器 2 收到: %d\n", n*2)
    }, "doubler")

    // 添加可能会 panic 的监听器
    numberSignal.AddListener(func(ctx context.Context, n int) {
        if n == 0 {
            panic("除数不能为 0")
        }
        fmt.Printf("监听器 3 收到: %d\n", 100/n)
    })

    // 发射信号
    ctx := context.Background()
    fmt.Println("发射信号 5")
    err := numberSignal.Emit(ctx, 5)
    if err != nil {
        fmt.Println("错误:", err)
    }

    fmt.Println("发射信号 0")
    err = numberSignal.Emit(ctx, 0)
    if err != nil {
        fmt.Println("错误:", err)
    }

    // 使用超时上下文
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()

    numberSignal.AddListener(func(ctx context.Context, n int) {
        time.Sleep(200 * time.Millisecond)
        fmt.Println("这条消息不会被打印，因为已经超时")
    }, "slow")

    fmt.Println("发射信号 10 (带超时)")
    err = numberSignal.Emit(ctx, 10)
    if err != nil {
        fmt.Println("错误:", err)
    }

    // 移除监听器
    numberSignal.RemoveListener("doubler")

    fmt.Println("发射信号 7 (移除了一个监听器)")
    err = numberSignal.Emit(context.Background(), 7)
    if err != nil {
        fmt.Println("错误:", err)
    }

    // 重置信号
    numberSignal.Reset()
    fmt.Printf("重置后的监听器数量: %d\n", numberSignal.Len())
}
```

## 7. 注意事项

1. 异步信号的监听器在单独的 goroutine 中执行，需要注意并发安全。
2. 在监听器中使用长时间运行的操作时，请确保正确处理上下文取消。
3. 虽然包会处理 panic，但最好在监听器中进行适当的错误处理，以避免不必要的 panic。
4. 移除监听器时，确保使用正确的键。如果在添加监听器时没有指定键，则无法单独移除该监听器。

## 8. 性能考虑

- 同步信号适用于需要按顺序处理且处理时间较短的场景。
- 异步信号适用于可以并行处理或处理时间较长的场景。
- 对于高频率的信号发射，考虑使用对象池来减少内存分配。

## 9. 调试技巧

1. 使用 `Len()` 方法检查当前的监听器数量。
2. 在添加监听器时使用有意义的键，这样可以更容易地跟踪和移除特定的监听器。
3. 在监听器中添加日志，以便跟踪信号的处理过程。

## 10. 结语

Signals 包提供了一个强大而灵活的信号系统，适用于各种事件驱动的编程场景。通过正确使用同步和异步信号、上下文控制和错误处理，您可以构建出健壮和高效的事件处理系统。

如果您在使用过程中遇到任何问题或有改进建议，请随时与我们联系。祝您使用愉快！