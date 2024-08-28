# 速率限制包使用手册

## 1. 简介

本包提供了一个灵活的令牌桶算法实现，用于进行速率限制。它支持两种模式：宽松模式和严格模式，可以根据不同的应用场景进行选择。

## 2. 主要特性

- 支持宽松和严格两种速率限制模式
- 基于令牌桶算法，提供平滑的速率控制
- 支持自定义填充间隔和桶容量
- 线程安全，可在并发环境中使用
- 支持上下文（context）用于超时和取消操作
- 提供了用于监控的指标收集功能

## 3. 引用

使用需要进行引用

```go
import (
    "github.com/sagoo-cloud/nexframe/utils/ratelimit"
)

```

## 4. 基本用法

### 4.1 创建令牌桶

```go
import (
    "context"
    "time"
    "github.com/sagoo-cloud/nexframe/utils/ratelimit"
)

// 创建一个每秒填充10个令牌的桶
ctx := context.Background()
bucket, err := ratelimit.NewBucket(ctx, time.Second, 10)
if err != nil {
    // 处理错误
}
```

### 4.2 使用令牌

```go
// 尝试获取一个令牌
err := bucket.Take(context.Background(), 1)
if err != nil {
    // 处理错误（例如，桶已关闭或上下文已取消）
}
```

### 4.3 关闭令牌桶

```go
bucket.Close()
```

## 5. 高级用法

### 5.1 使用严格模式

```go
bucket, err := ratelimit.NewBucket(ctx, time.Second, 10, ratelimit.WithStrictMode())
```

### 5.2 带超时的令牌获取

```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
defer cancel()

err := bucket.Take(ctx, 1)
if err != nil {
    // 处理错误（可能是超时或其他错误）
}
```

### 5.3 检查可用令牌数

```go
available := bucket.Available()
fmt.Printf("当前可用令牌数：%d\n", available)
```

## 6. 模式说明

### 6.1 宽松模式（默认）

- 允许在短时间内超过速率限制
- 适用于能够容忍短暂突发流量的场景
- 当没有可用令牌时，会引入一个小的延迟，但仍会允许操作执行

### 6.2 严格模式

- 严格遵守速率限制
- 当没有足够的令牌时，会阻塞直到有足够的令牌可用
- 适用于需要精确控制速率的场景

## 7. 示例

### 7.1 API 速率限制

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
    "github.com/sagoo-cloud/nexframe/utils/ratelimit"
)

var bucket *ratelimit.Bucket

func init() {
    var err error
    bucket, err = ratelimit.NewBucket(context.Background(), time.Second, 100)
    if err != nil {
        panic(err)
    }
}

func rateLimitMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        err := bucket.Take(r.Context(), 1)
        if err != nil {
            http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
            return
        }
        next.ServeHTTP(w, r)
    }
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", rateLimitMiddleware(handler))
    http.ListenAndServe(":8080", nil)
}
```

### 7.2 数据库查询限制

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"
    "github.com/sagoo-cloud/nexframe/utils/ratelimit"
)

var db *sql.DB
var queryBucket *ratelimit.Bucket

func init() {
    // 初始化数据库连接
    var err error
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        panic(err)
    }

    // 创建一个每秒允许10次查询的令牌桶
    queryBucket, err = ratelimit.NewBucket(context.Background(), time.Second/10, 1, ratelimit.WithStrictMode())
    if err != nil {
        panic(err)
    }
}

func rateLimit测Query(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error) {
    err := queryBucket.Take(ctx, 1)
    if err != nil {
        return nil, fmt.Errorf("rate limit exceeded: %w", err)
    }
    return db.QueryContext(ctx, query, args...)
}

func main() {
    ctx := context.Background()
    rows, err := rateLimit测Query(ctx, "SELECT * FROM users")
    if err != nil {
        fmt.Printf("查询错误: %v\n", err)
        return
    }
    defer rows.Close()
    // 处理查询结果
}
```

## 8. 注意事项

- 在高并发环境中，建议使用较大的桶容量和填充间隔，以减少锁竞争。
- 严格模式可能会导致请求阻塞，请确保设置适当的上下文超时。
- 定期监控 `Available()` 方法返回的可用令牌数，以帮助调整速率限制参数。
- 在分布式系统中，考虑使用集中式的速率限制服务，而不是单机限制。

## 9. 性能考虑

- 宽松模式通常具有更好的性能，因为它允许短时间的突发流量。
- 如果您的应用对延迟非常敏感，可以考虑使用宽松模式并通过其他机制（如监控和告警）来处理突发情况。

## 10. 故障排除

如果遇到问题，请检查以下几点：
- 确保正确设置了填充间隔和桶容量。
- 检查是否正确处理了 `Take()` 方法返回的错误。
- 在严格模式下，确保设置了适当的上下文超时。

如果问题仍然存在，请查看包的文档或联系维护者寻求帮助。