# 布隆过滤器

布隆过滤器是一种空间效率很高的概率型数据结构，用于判断一个元素是否属于一个集合。它可以快速地判断一个元素是否在集合中，但有一定的误判率。本实现使用 Redis 作为后端存储，适合分布式环境下的应用。

### 主要特性：

- 基于 Redis 的分布式布隆过滤器
- 可自定义哈希函数
- 支持元素添加和查询操作
- 可配置过期时间
- 线程安全

## 2. 安装

确保您的 Go 环境已经正确设置，然后运行以下命令安装包：

```bash
go get github.com/sagoo-cloud/nexframe
```

## 3. 基本用法

以下是一个简单的使用示例：

```go
package main

import (
	"context"
	"fmt"
	"github.com/redis/go-redis/v9"
	"github.com/sagoo-cloud/nexframe/os/bloom"
)

func main() {
	// 创建 Redis 客户端
	redisClient := redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})

	// 创建布隆过滤器实例
	bf, err := bloom.New(
		bloom.WithRedis(redisClient), //如果不设置将使用框架默认的
		bloom.WithKey("my-bloom-filter"),
	)
	if err != nil {
		panic(err)
	}

	ctx := context.Background()

	// 添加元素
	err = bf.Add(ctx, "apple", "banana", "cherry")
	if err != nil {
		panic(err)
	}

	// 检查元素是否存在
	exists, err := bf.Exist(ctx, "apple")
	if err != nil {
		panic(err)
	}
	fmt.Printf("Does 'apple' exist? %v\n", exists)

	exists, err = bf.Exist(ctx, "durian")
	if err != nil {
		panic(err)
	}
	fmt.Printf("Does 'durian' exist? %v\n", exists)
}
```

## 4. 配置选项

布隆过滤器提供了多个配置选项，可以在创建实例时使用：

### WithRedis(redis.UniversalClient)

设置 Redis 客户端。如果不设置将采用框架默认的redis配置。

```go
redisClient := redis.NewClient(&redis.Options{
	Addr: "localhost:6379",
})
bf, err := bloom.New(bloom.WithRedis(redisClient))
```

### WithKey(string)

设置 Redis 中使用的键名。

```go
bf, err := bloom.New(bloom.WithKey("my-custom-key"))
```

### WithExpire(time.Duration)

设置过滤器的过期时间。

```go
bf, err := bloom.New(bloom.WithExpire(1 * time.Hour))
```

### WithHash(...func(string) uint64)

添加自定义哈希函数。

```go
customHash := func(s string) uint64 {
	h := fnv.New64a()
	h.Write([]byte(s))
	return h.Sum64()
}
bf, err := bloom.New(bloom.WithHash(customHash))
```

### WithTimeout(time.Duration)

设置操作超时时间。

```go
bf, err := bloom.New(bloom.WithTimeout(5 * time.Second))
```

## 5. 高级用法

### 使用自定义哈希函数

您可以实现自己的哈希函数来优化布隆过滤器的性能或降低冲突率：

```go
import (
	"github.com/spaolacci/murmur3"
	"github.com/your-username/bloom-filter"
)

func murmurHash(s string) uint64 {
	return murmur3.Sum64([]byte(s))
}

bf, err := bloom.New(
	bloom.WithHash(murmurHash),
	bloom.WithRedis(redisClient),
)
```

### 批量添加元素

对于需要添加大量元素的场景，可以使用批量添加来提高性能：

```go
elements := []string{"apple", "banana", "cherry", "date", "elderberry"}
err := bf.Add(ctx, elements...)
if err != nil {
	// 处理错误
}
```

### 并发使用

布隆过滤器的方法是线程安全的，可以在并发环境中使用：

```go
var wg sync.WaitGroup
for i := 0; i < 100; i++ {
	wg.Add(1)
	go func(i int) {
		defer wg.Add(-1)
		element := fmt.Sprintf("element-%d", i)
		err := bf.Add(ctx, element)
		if err != nil {
			fmt.Printf("Error adding element %s: %v\n", element, err)
		}
	}(i)
}
wg.Wait()
```

## 6. 性能考虑

- **选择合适的哈希函数**：默认提供的哈希函数（BKDRHash、SDBMHash、DJBHash）通常表现良好，但对于特定的数据集，自定义哈希函数可能会带来性能提升。

- **调整过期时间**：根据您的使用场景，合理设置过期时间可以平衡内存使用和数据新鲜度。

- **批量操作**：当需要添加或查询大量元素时，尽可能使用批量操作来减少网络往返。

- **监控 Redis 性能**：布隆过滤器的性能很大程度上取决于 Redis 的性能。定期监控 Redis 的内存使用、响应时间等指标。

## 7. 常见问题解答

**Q: 布隆过滤器的误判率是多少？**

A: 误判率取决于过滤器的大小、哈希函数的数量以及已添加的元素数量。您可以通过增加位数组的大小或使用更多的哈希函数来降低误判率，但这会增加内存使用和计算开销。

**Q: 如何选择合适的哈希函数？**

A: 好的哈希函数应该具有均匀分布的特性，并且计算速度快。默认提供的哈希函数通常足够好，但如果您有特殊需求，可以考虑使用如 MurmurHash、xxHash 等现代哈希算法。

**Q: 布隆过滤器支持删除元素吗？**

A: 标准的布隆过滤器不支持删除元素。如果需要支持删除操作，可以考虑使用计数布隆过滤器（Counting Bloom Filter）的变体。

**Q: 如何处理布隆过滤器的扩容？**

A: 本实现不直接支持动态扩容。如果需要处理更多元素，建议创建一个新的、更大的过滤器，并重新添加所有元素。

**Q: 在分布式环境中如何使用这个布隆过滤器？**

A: 由于使用 Redis 作为后端存储，这个布隆过滤器天然支持分布式环境。只需确保所有节点使用相同的 Redis 实例或集群即可。

如果您有任何其他问题或需要进一步的说明，请随时询问。祝您使用愉快！