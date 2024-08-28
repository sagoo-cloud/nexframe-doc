# Cache 包开发手册

## 1. 简介

Cache 包是一个高性能的缓存解决方案，结合了内存缓存和 Redis 缓存的优势。它提供了简单易用的 API，用于存储和检索各种类型的数据，包括字节数组、字符串和结构体。

## 2. 安装

确保你的项目中已经导入了 Cache 包：

```go
import "github.com/sagoo-cloud/nexframe/os/cache"
```

## 3. 配置

在使用 Cache 包之前，你需要创建一个配置对象：

```go
config := &cache.Config{
    MemoryCacheSize: 100 * 1024 * 1024, // 100MB
    RedisAddr:       "localhost:6379",
    RedisPassword:   "",
    RedisDB:         0,
    RedisPrefix:     "myapp:",
}
```

## 4. 初始化

使用配置对象创建一个 CacheManager 实例：

```go
cacheManager := cache.NewCacheManager(config)
```

## 5. 基本操作

### 5.1 设置缓存

```go
err := cacheManager.Set("key", []byte("value"), time.Minute)
if err != nil {
    log.Printf("设置缓存失败: %v", err)
}
```

### 5.2 获取缓存

```go
value, exists, err := cacheManager.Get("key")
if err != nil {
    log.Printf("获取缓存失败: %v", err)
} else if exists {
    fmt.Printf("缓存值: %s\n", string(value))
} else {
    fmt.Println("缓存不存在")
}
```

### 5.3 删除缓存

```go
err := cacheManager.Delete("key")
if err != nil {
    log.Printf("删除缓存失败: %v", err)
}
```

## 6. 高级功能

### 6.1 存储结构体

Cache 包支持直接存储和检索结构体：

```go
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
    Age  int    `json:"age"`
}

user := User{ID: 1, Name: "Alice", Age: 30}

// 存储结构体
err := cacheManager.SetStruct("user:1", user, time.Hour)
if err != nil {
    log.Printf("存储用户失败: %v", err)
}

// 检索结构体
var retrievedUser User
exists, err := cacheManager.GetStruct("user:1", &retrievedUser)
if err != nil {
    log.Printf("获取用户失败: %v", err)
} else if exists {
    fmt.Printf("检索到的用户: %+v\n", retrievedUser)
} else {
    fmt.Println("用户不存在")
}
```

### 6.2 获取键列表

你可以获取缓存中所有键或指定前缀的键：

```go
// 获取所有键
allKeys, err := cacheManager.Keys("")
if err != nil {
    log.Printf("获取所有键失败: %v", err)
} else {
    fmt.Println("所有键:", allKeys)
}

// 获取指定前缀的键
userKeys, err := cacheManager.Keys("user:")
if err != nil {
    log.Printf("获取用户键失败: %v", err)
} else {
    fmt.Println("用户键:", userKeys)
}
```

## 7. 错误处理

Cache 包提供了自定义错误处理的能力：

```go
type CustomErrorHandler struct{}

func (h *CustomErrorHandler) HandleError(err error) {
    log.Printf("缓存错误: %v", err)
}

cacheManager.WithErrorHandler(&CustomErrorHandler{})
```

## 8. 性能优化

### 8.1 预热缓存

对于频繁访问的数据，你可以使用预热功能来提高性能：

```go
keys := []string{"user:1", "user:2", "product:1"}
err := cacheManager.PrewarmCache(keys)
if err != nil {
    log.Printf("预热缓存失败: %v", err)
}
```

### 8.2 批量操作

对于需要同时操作多个键值对的场景，可以使用批量操作来提高效率：

```go
// 批量设置
items := map[string][]byte{
    "key1": []byte("value1"),
    "key2": []byte("value2"),
    "key3": []byte("value3"),
}
err := cacheManager.BatchSet(items, time.Minute)
if err != nil {
    log.Printf("批量设置失败: %v", err)
}

// 批量获取
keys := []string{"key1", "key2", "key3"}
results, err := cacheManager.BatchGet(keys)
if err != nil {
    log.Printf("批量获取失败: %v", err)
} else {
    for key, value := range results {
        fmt.Printf("键: %s, 值: %s\n", key, string(value))
    }
}
```

## 9. 注意事项

1. 内存使用：注意监控内存缓存的使用情况，避免占用过多内存。

2. Redis 连接：确保 Redis 服务器可用，并正确配置连接参数。

3. 键冲突：在多个应用共享同一个 Redis 实例时，使用适当的前缀避免键冲突。

4. 序列化：存储复杂对象时，注意可能的序列化/反序列化错误。

5. 过期时间：合理设置缓存项的过期时间，避免缓存数据过期或占用过多空间。

## 10. 示例

以下是一个完整的示例，展示了如何使用 Cache 包的主要功能：

```go
package main

import (
    "fmt"
    "log"
    "time"
    "yourproject/cache"
)

type Product struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Price float64 `json:"price"`
}

func main() {
    config := &cache.Config{
        MemoryCacheSize: 100 * 1024 * 1024,
        RedisAddr:       "localhost:6379",
        RedisPassword:   "",
        RedisDB:         0,
        RedisPrefix:     "myapp:",
    }

    cacheManager := cache.NewCacheManager(config)

    // 设置简单的键值对
    err := cacheManager.Set("greeting", []byte("Hello, World!"), time.Minute)
    if err != nil {
        log.Printf("设置greeting失败: %v", err)
    }

    // 获取简单的键值对
    greeting, exists, err := cacheManager.Get("greeting")
    if err != nil {
        log.Printf("获取greeting失败: %v", err)
    } else if exists {
        fmt.Printf("Greeting: %s\n", string(greeting))
    }

    // 存储结构体
    product := Product{ID: 1, Name: "Laptop", Price: 999.99}
    err = cacheManager.SetStruct("product:1", product, time.Hour)
    if err != nil {
        log.Printf("存储产品失败: %v", err)
    }

    // 检索结构体
    var retrievedProduct Product
    exists, err = cacheManager.GetStruct("product:1", &retrievedProduct)
    if err != nil {
        log.Printf("获取产品失败: %v", err)
    } else if exists {
        fmt.Printf("检索到的产品: %+v\n", retrievedProduct)
    }

    // 获取所有键
    allKeys, err := cacheManager.Keys("")
    if err != nil {
        log.Printf("获取所有键失败: %v", err)
    } else {
        fmt.Println("所有键:", allKeys)
    }

    // 删除键
    err = cacheManager.Delete("greeting")
    if err != nil {
        log.Printf("删除greeting失败: %v", err)
    }

    // 再次检查键是否存在
    _, exists, _ = cacheManager.Get("greeting")
    fmt.Printf("greeting是否仍然存在: %v\n", exists)
}
```

这个示例展示了如何设置和获取简单的键值对、如何存储和检索结构体、如何获取所有键以及如何删除键。
