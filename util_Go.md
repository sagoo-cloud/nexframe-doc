# util.Go 函数使用手册

## 简介

`util.Go` 是一个强大的工具函数，用于在 Go 语言中安全地启动 goroutine。它提供了错误处理和 panic 恢复机制，使得并发编程更加安全和可控。

## 功能特点

1. 自动错误处理和 panic 恢复
2. 支持自定义错误处理函数
3. 提供默认的错误日志记录
4. 简化并发编程，提高代码可靠性

## 函数签名

```go
func Go(fn func() error, errHandler ...ErrHandler)
```

参数说明：
- `fn`: 要在新 goroutine 中执行的函数，必须返回一个 error。
- `errHandler`: 可选的自定义错误处理函数。

## 使用方法

### 基本用法

最简单的使用方式是只传入要执行的函数：

```go
util.Go(func() error {
    // 你的代码逻辑
    return nil
})
```

这种方式会使用默认的错误处理器，它会将错误信息记录到标准日志中。

### 使用自定义错误处理器

如果你需要自定义错误处理逻辑，可以传入第二个参数：

```go
util.Go(
    func() error {
        // 你的代码逻辑
        return errors.New("一个示例错误")
    },
    func(err interface{}, stack []byte) {
        log.Printf("发生错误：%v\n堆栈：%s", err, stack)
    }
)
```

### 处理可能发生的 panic

`util.Go` 会自动捕获并处理 panic：

```go
util.Go(func() error {
    panic("发生了一个 panic")
    return nil
})
```

在这种情况下，panic 会被捕获，并通过错误处理器处理。

## 高级用法

### 在错误处理器中区分错误和 panic

错误处理器接收两个参数：`err` 和 `stack`。如果发生 panic，`stack` 会包含堆栈信息；如果是普通错误，`stack` 将为 nil。

```go
util.Go(
    func() error {
        // 你的代码逻辑
        return nil
    },
    func(err interface{}, stack []byte) {
        if stack != nil {
            fmt.Printf("发生了 panic：%v\n堆栈：%s", err, stack)
        } else {
            fmt.Printf("发生了错误：%v", err)
        }
    }
)
```

### 使用 context 进行取消操作

虽然 `util.Go` 本身不直接支持 context，但你可以在传入的函数中使用 context：

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

util.Go(func() error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case <-time.After(10 * time.Second):
        fmt.Println("操作完成")
    }
    return nil
})
```

## 最佳实践

1. 始终处理返回的错误，即使你认为不会发生错误。
2. 在处理长时间运行的任务时，考虑使用 context 进行超时控制。
3. 避免在 `util.Go` 中启动的函数里再次使用 `go` 关键字启动 goroutine，而应该嵌套使用 `util.Go`。
4. 在错误处理器中，根据错误的严重程度选择适当的日志级别（如 info、warning、error）。

## 注意事项

1. `util.Go` 不会等待 goroutine 完成。如果需要等待，请使用 `sync.WaitGroup`。
2. 虽然 `util.Go` 可以捕获 panic，但应尽量避免编写可能导致 panic 的代码。
3. 大量使用 `util.Go` 可能会导致性能轻微下降，因为它比原生 `go` 关键字增加了一些开销。

