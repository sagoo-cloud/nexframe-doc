# 幂等性检查

该组件、提供了一个基于 Redis 的幂等性检查实现。它可以用于确保特定操作只执行一次，即使该操作被多次调用。这在分布式系统中特别有用，可以防止重复处理和数据不一致。

## 2. 安装

确保你的项目中已经安装了 Go-Redis 客户端：

```bash
go get github.com/redis/go-redis/v9
```

然后，将本程序的代码添加到你的项目中。

## 3. 基本用法

### 3.1 创建 Idempotent 实例

首先，你需要创建一个 `Idempotent` 实例：

```go
import (
    "github.com/redis/go-redis/v9"
    "github.com/sagoo-cloud/nexframe/os/idempotent"
)

// 创建 Redis 客户端
redisClient := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
})

// 创建 Idempotent 实例
idem := idempotent.New(
    idempotent.WithRedis(redisClient),//如果不设置，将采用框架中redis的默认配置
    idempotent.WithPrefix("my_app"),
    idempotent.WithExpire(30), // 30 分钟过期
)
```

### 3.2 生成 Token

在执行需要幂等性保证的操作之前，生成一个 token：

```go
ctx := context.Background()
token, err := idem.Token(ctx)
if err != nil {
    // 处理错误
}
```

### 3.3 检查 Token

在执行操作之前，检查 token 是否有效：

```go
pass, err := idem.Check(ctx, token)
if err != nil {
    // 处理错误
}
if !pass {
    // token 无效或已使用，不执行操作
    return
}

// 执行需要幂等性保证的操作
```

## 4. 高级配置

### 4.1 自定义前缀

你可以为你的 token 设置自定义前缀，以避免与其他应用冲突：

```go
idem := idempotent.New(
    idempotent.WithPrefix("my_custom_prefix"),
)
```

### 4.2 自定义过期时间

设置 token 的过期时间（单位：分钟）：

```go
idem := idempotent.New(
    idempotent.WithExpire(60), // 60 分钟后过期
)
```

## 5. 最佳实践

1. **错误处理**：总是检查 `Token()` 和 `Check()` 方法返回的错误。

2. **合理设置过期时间**：根据你的业务需求设置合适的过期时间。太短可能导致正常操作被拒绝，太长可能无法有效防止重复操作。

3. **在事务中使用**：在数据库事务中使用 `Check()` 方法，以确保操作的原子性。

4. **日志记录**：记录所有的 token 生成和检查操作，以便后续调试和审计。

5. **监控**：监控 Redis 的性能和可用性，因为它对于幂等性检查至关重要。

## 6. 注意事项

- 确保 Redis 服务器可用且配置正确。
- 在分布式环境中，确保所有节点使用相同的 Redis 实例。
- 定期清理过期的 token，以节省 Redis 存储空间。

## 7. 故障排除

1. 如果 `Token()` 或 `Check()` 返回 `ErrRedisNotEnabled`，检查 Redis 客户端是否正确初始化。

2. 如果操作执行速度变慢，检查 Redis 的性能和网络连接。

3. 如果出现意外的重复操作，检查是否有其他系统或进程在使用相同的 token。

## 8. 示例代码

```go
package main

import (
    "context"
    "fmt"
    "github.com/redis/go-redis/v9"
    "github.com/sagoo-cloud/nexframe/os/idempotent"
)

func main() {
    redisClient := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })

    idem := idempotent.New(
        idempotent.WithRedis(redisClient),
        idempotent.WithPrefix("example"),
        idempotent.WithExpire(30),
    )

    ctx := context.Background()

    // 生成 token
    token, err := idem.Token(ctx)
    if err != nil {
        fmt.Printf("生成 token 失败: %v\n", err)
        return
    }

    // 检查 token
    pass, err := idem.Check(ctx, token)
    if err != nil {
        fmt.Printf("检查 token 失败: %v\n", err)
        return
    }

    if pass {
        fmt.Println("Token 有效，执行操作")
        // 执行你的业务逻辑
    } else {
        fmt.Println("Token 无效或已使用，不执行操作")
    }
}
```

希望这份使用手册能帮助你更好地使用这个幂等性检查程序。如有任何问题，请随时咨询。