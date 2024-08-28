# GPool 使用手册

## 1. 简介

GPool 是一个高效的 Go 语言协程池实现，旨在管理和重用 goroutine，以提高性能并控制并发。它特别适用于需要处理大量并发任务的应用程序，如 Web 服务器、数据处理管道或后台作业系统。

## 2. 主要特性

- 可控的并发度
- 自动任务队列管理
- 支持上下文取消
- 优雅的关闭机制
- 提供运行中任务数量的实时统计

## 3. 安装

在你的 Go 项目中使用以下命令安装 GPool：

```bash
go get github.com/your-username/gpool
```

## 4. 基本用法

### 4.1 创建 GPool

```go
import "github.com/your-username/gpool"

// 创建一个容量为 10 的协程池
pool := gpool.NewGPool(10)
```

### 4.2 提交任务

GPool 提供两种方法来提交任务：`Go` 和 `AddJob`。

#### 使用 Go 方法

`Go` 方法会尝试立即执行任务，如果池已满，则将任务加入队列。

```go
pool.Go(func(ctx context.Context) error {
    // 执行任务
    return nil
})
```

#### 使用 AddJob 方法

`AddJob` 方法会将任务添加到队列中，如果队列已满，它会阻塞直到有空间可用。

```go
pool.AddJob(func(ctx context.Context) error {
    // 执行任务
    return nil
})
```

### 4.3 等待任务完成

使用 `Wait` 方法等待所有提交的任务完成：

```go
pool.Wait()
```

### 4.4 关闭池

使用 `Shutdown` 方法优雅地关闭池：

```go
pool.Shutdown()
```

### 4.5 获取运行中的任务数

使用 `RunningCount` 方法获取当前正在运行的任务数：

```go
count := pool.RunningCount()
fmt.Printf("当前运行的任务数：%d\n", count)
```

## 5. 高级用法

### 5.1 使用上下文控制任务

GPool 支持通过上下文来控制任务的生命周期：

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

pool.Go(func(jobCtx context.Context) error {
    select {
    case <-jobCtx.Done():
        fmt.Println("任务被取消")
        return jobCtx.Err()
    case <-time.After(3 * time.Second):
        fmt.Println("任务完成")
        return nil
    }
})
```

### 5.2 错误处理

任务函数可以返回错误，你应该在任务中妥善处理这些错误：

```go
pool.Go(func(ctx context.Context) error {
    // 模拟可能失败的操作
    if rand.Float32() < 0.3 {
        return errors.New("随机错误")
    }
    return nil
})
```

## 6. 最佳实践

1. **合理设置池容量**：池容量应根据你的系统资源和预期负载来设置。

2. **使用 defer 关闭池**：在 main 函数或长期运行的程序中，使用 defer 确保池被正确关闭。

3. **妥善处理错误**：始终检查并处理任务返回的错误。

4. **避免长时间运行的任务**：将长时间运行的任务分解为较小的子任务，以提高池的整体吞吐量。

5. **使用上下文控制任务生命周期**：特别是对于可能需要取消的长时间运行任务。

## 7. 完整示例

以下是一个使用 GPool 的完整示例：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/rand"
    "time"

    "github.com/your-username/gpool"
)

func main() {
    // 创建一个容量为 5 的协程池
    pool := gpool.NewGPool(5)
    defer pool.Shutdown()

    // 创建一个带有超时的上下文
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    // 提交 20 个任务
    for i := 0; i < 20; i++ {
        taskID := i
        pool.Go(func(jobCtx context.Context) error {
            // 模拟工作负载
            time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)

            select {
            case <-jobCtx.Done():
                fmt.Printf("任务 %d 被取消\n", taskID)
                return jobCtx.Err()
            default:
                fmt.Printf("任务 %d 完成\n", taskID)
                return nil
            }
        })
    }

    // 等待所有任务完成或上下文取消
    <-ctx.Done()

    // 等待池中的所有任务完成
    pool.Wait()

    fmt.Printf("总共运行的任务数：%d\n", pool.RunningCount())
}
```

## 8. 注意事项

- GPool 不保证任务的执行顺序。
- 如果池已满且工作队列也已满，`Go` 方法会在新的 goroutine 中执行任务，这可能会暂时增加系统负载。
- `Shutdown` 方法会等待所有任务完成，在此期间不应再提交新任务。

## 9. 性能考虑

GPool 设计用于高并发场景，但实际性能可能因使用方式和系统资源而异。在生产环境中使用前，建议进行充分的性能测试和负载测试。

## 

GPool 提供了一种简单而强大的方式来管理 Go 程序中的并发任务。通过合理使用 GPool，你可以有效控制程序的并发度，提高资源利用率，并简化错误处理和任务管理。

