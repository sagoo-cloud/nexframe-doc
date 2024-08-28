# 分布式锁

## 1. 简介

这个包是一个基于 Redis 的分布式锁库，用于在分布式系统中实现互斥访问共享资源。这个库提供了简单而强大的 API，允许开发者轻松地在其应用程序中实现分布式锁定机制。

## 2. 安装

首先，确保你的项目中已经安装了 Redis 客户端库。你可以使用以下命令安装：

```
go get github.com/redis/go-redis/v9
```

然后，将 Nx 库添加到你的项目中（假设 Nx 库已经发布）：

```
go get github.com/sagoo-cloud/nexframe
```

## 3. 基本用法

### 3.1 创建 Nx 实例

要使用 Nx，首先需要创建一个 Nx 实例。以下是一个基本的示例：

```go
import (
    "github.com/redis/go-redis/v9"
    "github.com/sagoo-cloud/nexframe/os/nx"
)

func main() {
    // 创建 Redis 客户端
    redisClient := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })

    // 创建 Nx 实例
    nxLock, err := nx.New(
        nx.WithRedis(redisClient),
        nx.WithKey("my-lock-key"),
    )
    if err != nil {
        panic(err)
    }

    // 使用 nxLock ...
}
```

### 3.2 获取锁

使用 `Lock` 方法尝试获取锁：

```go
ctx := context.Background()
err := nxLock.Lock(ctx)
if err != nil {
    // 处理错误
    return
}
defer nxLock.Unlock(ctx)

// 执行需要锁保护的操作
```

### 3.3 释放锁

锁会在 `Unlock` 方法调用时释放，或者在达到过期时间后自动释放。

```go
err := nxLock.Unlock(ctx)
if err != nil {
    // 处理错误
}
```

## 4. 高级配置

Nx 提供了多个选项来自定义锁的行为：

### 4.1 设置过期时间

```go
nxLock, err := nx.New(
    nx.WithRedis(redisClient),
    nx.WithKey("my-lock-key"),
    nx.WithExpire(30), // 设置锁的过期时间为 30 秒
)
```

### 4.2 配置重试策略

```go
nxLock, err := nx.New(
    nx.WithRedis(redisClient),
    nx.WithKey("my-lock-key"),
    nx.WithRetry(5),                    // 设置最大重试次数为 5
    nx.WithInterval(50 * time.Millisecond), // 设置重试间隔为 50 毫秒
)
```

## 5. 最佳实践

1. **合理设置过期时间**：过期时间应该足够长，以覆盖预期的操作时间，但又不能太长，以防止在出现问题时长时间锁定资源。

2. **使用 defer 释放锁**：总是使用 `defer` 来确保锁被释放，即使发生panic。

3. **处理获取锁的超时**：使用带超时的 context 来控制获取锁的最长等待时间。

   ```go
   ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
   defer cancel()
   
   err := nxLock.Lock(ctx)
   if err != nil {
       if err == context.DeadlineExceeded {
           // 获取锁超时
       } else {
           // 其他错误
       }
       return
   }
   defer nxLock.Unlock(context.Background())
   ```

4. **避免长时间持有锁**：尽量减少锁内的操作时间，只锁定真正需要互斥的代码段。

## 6. 错误处理

Nx 可能返回以下错误：

- `lock timeout`：当超过最大重试次数仍未获得锁时。
- `context.DeadlineExceeded`：当 context 超时时。
- Redis 相关错误：当与 Redis 通信出现问题时。

始终检查并适当处理这些错误。

## 7. 性能考虑

- Nx 使用 Redis 的 EVAL 命令来确保获取锁的原子性，这在大多数情况下性能良好。
- 如果你的应用程序对延迟特别敏感，可以考虑调整重试间隔和最大重试次数。

## 8. 限制和注意事项

- Nx 依赖于 Redis 的可用性。确保你的 Redis 设置有适当的冗余和故障转移机制。
- 虽然 Nx 实现了基本的死锁预防（通过过期时间），但在分布式系统中仍然需要谨慎处理各种边缘情况。

## 9. 示例：在 Web 服务中使用 Nx

以下是一个在 Web 服务中使用 Nx 的完整示例：

```go
package main

import (
    "context"
    "github.com/redis/go-redis/v9"
    "github.com/sagoo-cloud/nexframe/os/nx"
    "log"
    "net/http"
    "time"
)

var nxLock *nx.Nx

func init() {
    redisClient := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })

    var err error
    nxLock, err = nx.New(
        nx.WithRedis(redisClient),
        nx.WithKey("critical-section-lock"),
        nx.WithExpire(30),
        nx.WithRetry(5),
        nx.WithInterval(100 * time.Millisecond),
    )
    if err != nil {
        log.Fatalf("Failed to create Nx lock: %v", err)
    }
}

func criticalSection(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()

    err := nxLock.Lock(ctx)
    if err != nil {
        if err == context.DeadlineExceeded {
            http.Error(w, "Failed to acquire lock: timeout", http.StatusServiceUnavailable)
        } else {
            http.Error(w, "Failed to acquire lock", http.StatusInternalServerError)
        }
        log.Printf("Lock acquisition failed: %v", err)
        return
    }
    defer nxLock.Unlock(context.Background())

    // 执行需要锁保护的操作
    time.Sleep(2 * time.Second) // 模拟一些工作
    w.Write([]byte("Critical section accessed successfully"))
}

func main() {
    http.HandleFunc("/critical", criticalSection)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

这个示例展示了如何在 Web 服务的特定路由中使用 Nx 锁来保护临界区。它包含了错误处理、超时设置和适当的锁释放机制。

## 10. 结论

Nx 提供了一个简单而强大的方式来在分布式系统中实现互斥。通过遵循本手册中的最佳实践和建议，你可以有效地在你的 Go 应用程序中使用分布式锁，确保共享资源的安全访问。