# 定时器服务器包使用手册

## 1. 概述

定时器服务器包是一个用于管理和执行周期性任务的 Go 语言库。它提供了一个灵活、高效且线程安全的方式来处理需要定期执行的任务。这个包特别适用于需要在后台周期性运行任务的应用程序，如数据同步、清理操作、状态更新等。

## 2. 主要特性

- 支持注册多个具有不同执行频率的定时任务
- 线程安全的操作，支持并发环境
- 优雅的关闭机制，确保所有任务完成后再退出
- 使用 context 进行超时控制和取消操作
- 内置日志记录功能

## 3. 安装

使用以下命令安装包：

```
go get github.com/sagoo-cloud/nexframe
```

请将 `your-username` 替换为实际的 GitHub 用户名或包的实际路径。

## 4. 使用方法

### 4.1 创建服务器实例

首先，需要创建一个 `Server` 实例：

```go
import (
    "github.com/sagoo-cloud/nexframe/servers/timers"
    "log"
    "os"
)

logger := log.New(os.Stdout, "TIMER: ", log.Ldate|log.Ltime|log.Lshortfile)
server := timers.NewServer(logger)
```

### 4.2 注册服务

使用 `Register` 方法注册定时任务：

```go
err := server.Register("taskName", time.Second*30, handler, params)
if err != nil {
    log.Printf("注册任务失败: %v", err)
}
```

参数说明：
- `taskName`: 任务的唯一标识符
- `time.Second*30`: 任务执行频率（在这个例子中是每30秒）
- `handler`: 任务的处理函数
- `params`: 传递给处理函数的参数（可以为 nil）

### 4.3 定义处理函数

处理函数必须符合以下签名：

```go
func(context.Context, map[string]interface{}) (interface{}, error)
```

例如：

```go
func myHandler(ctx context.Context, params map[string]interface{}) (interface{}, error) {
    // 执行任务逻辑
    return "任务完成", nil
}
```

### 4.4 启动服务

注册完所有任务后，使用 `Run` 方法启动服务：

```go
err := server.Run()
if err != nil {
    log.Fatalf("启动服务失败: %v", err)
}
```

### 4.5 关闭服务

要优雅地关闭服务，使用 `Close` 方法：

```go
err := server.Close()
if err != nil {
    log.Printf("关闭服务器时发生错误: %v", err)
}
```

## 5. 完整示例

以下是一个完整的使用示例：

```go
package main

import (
    "context"
    "github.com/sagoo-cloud/nexframe/servers/timers"
    "log"
    "os"
    "time"
)

func main() {
    // 创建日志记录器
    logger := log.New(os.Stdout, "TIMER: ", log.Ldate|log.Ltime|log.Lshortfile)

    // 创建服务器实例
    server := timers.NewServer(logger)

    // 注册任务
    err := server.Register("task1", time.Second*10, task1Handler, nil)
    if err != nil {
        log.Fatalf("注册任务1失败: %v", err)
    }

    err = server.Register("task2", time.Second*20, task2Handler, map[string]interface{}{"key": "value"})
    if err != nil {
        log.Fatalf("注册任务2失败: %v", err)
    }

    // 启动服务
    err = server.Run()
    if err != nil {
        log.Fatalf("启动服务失败: %v", err)
    }

    // 让服务运行一段时间
    time.Sleep(time.Minute)

    // 关闭服务
    err = server.Close()
    if err != nil {
        log.Printf("关闭服务器时发生错误: %v", err)
    }
}

func task1Handler(ctx context.Context, params map[string]interface{}) (interface{}, error) {
    log.Println("执行任务1")
    return "任务1完成", nil
}

func task2Handler(ctx context.Context, params map[string]interface{}) (interface{}, error) {
    log.Printf("执行任务2，参数: %v", params)
    return "任务2完成", nil
}
```

## 6. 注意事项

1. 确保为每个任务指定唯一的名称，重复的名称会导致注册失败。
2. 处理函数应该是非阻塞的，或者使用提供的 context 来实现超时控制。
3. 如果任务执行时间超过了指定的频率，下一次执行会在当前执行完成后立即开始。
4. 在生产环境中，建议使用更复杂的日志记录方式，例如写入文件或发送到日志服务。

## 7. 故障排除

1. 如果任务没有按预期执行，检查是否正确调用了 `Serve()` 方法。
2. 如果遇到并发问题，确保没有在多个 goroutine 中同时修改共享状态。
3. 如果服务无法正常关闭，检查是否有长时间运行的任务阻止了关闭过程。

## 8. 性能考虑

- 对于需要高频率执行的短任务，考虑使用更低的执行间隔。
- 对于资源密集型任务，考虑增加执行间隔以减少系统负载。
- 在高并发环境中，可能需要限制同时运行的任务数量以避免资源耗尽。

