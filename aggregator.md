# Aggregator数据聚合器

## 1. 概述

Aggregator模块是一个用于批量处理数据的高性能并发组件。它能够将输入的数据项聚合成批次，然后异步处理这些批次，同时提供了灵活的配置选项和错误处理机制。

## 2. 主要结构

### 2.1 Aggregator

`Aggregator` 是这个模块的核心结构，它包含以下主要字段：

- `option`: 聚合器配置选项
- `eventQueue`: 事件队列，用于存储待处理的数据项
- `batchProcessor`: 批处理函数
- `pool`: 对象池，用于复用批次切片
- `lingerTimer`: 延迟处理计时器
- `lastProcessTime`: 上次处理时间

### 2.2 AggregatorOption

`AggregatorOption` 结构体用于配置Aggregator，包括以下字段：

- `BatchSize`: 批处理大小
- `Workers`: 工作协程数量
- `ChannelBufferSize`: 通道缓冲区大小
- `LingerTime`: 延迟处理时间
- `ErrorHandler`: 错误处理函数
- `Logger`: 日志记录器

## 3. 初始化和配置

### 3.1 创建新的Aggregator实例

使用 `NewAggregator` 函数创建新的Aggregator实例：

```go
aggregator, err := database.NewAggregator(batchProcessFunc, optionFuncs...)
```

### 3.2 配置选项

使用以下函数来设置Aggregator的配置选项：

- `WithBatchSize(size int)`: 设置批处理大小
- `WithWorkers(workers int)`: 设置工作协程数量
- `WithChannelBufferSize(size int)`: 设置通道缓冲区大小
- `WithLingerTime(duration time.Duration)`: 设置延迟处理时间
- `WithLogger(logger *log.Logger)`: 设置日志记录器
- `WithErrorHandler(handler ErrorHandlerFunc)`: 设置错误处理函数

示例：

```go
aggregator, err := database.NewAggregator(
    batchProcessFunc,
    database.WithBatchSize(10),
    database.WithWorkers(4),
    database.WithLingerTime(time.Second * 30),
)
```

## 4. 主要功能

### 4.1 数据入队

#### 4.1.1 非阻塞入队

```go
success := aggregator.TryEnqueue(item)
```

#### 4.1.2 阻塞入队

```go
err := aggregator.Enqueue(item)
```

#### 4.1.3 带重试的入队

```go
success := aggregator.EnqueueWithRetry(item, maxRetries, backoff)
```

### 4.2 启动和停止

#### 4.2.1 启动Aggregator

```go
aggregator.Start()
```

#### 4.2.2 停止Aggregator

```go
aggregator.Stop()
```

#### 4.2.3 安全停止Aggregator

```go
aggregator.SafeStop()
```

## 5. 批处理函数

批处理函数是Aggregator的核心，它定义了如何处理一批数据项：

```go
type BatchProcessFunc func([]interface{}) error
```

示例：

```go
func batchProcessFunc(items []interface{}) error {
    // 处理一批数据项
    for _, item := range items {
        // 处理单个数据项
    }
    return nil
}
```

## 6. 错误处理

错误处理函数允许自定义如何处理批处理过程中的错误：

```go
type ErrorHandlerFunc func(err error, items []interface{}, batchProcessFunc BatchProcessFunc, aggregator *Aggregator)
```

示例：

```go
func errorHandler(err error, items []interface{}, batchProcessFunc BatchProcessFunc, aggregator *Aggregator) {
    log.Printf("处理错误: %v", err)
    // 可以选择重试、跳过或其他处理方式
}
```

## 7. 最佳实践

1. 根据实际需求调整批处理大小和工作协程数量，以平衡吞吐量和资源使用。
2. 使用 `SafeStop()` 来确保所有数据都被处理后再停止Aggregator。
3. 实现适当的错误处理函数，以便在批处理失败时采取合适的措施。
4. 使用日志记录器来监控Aggregator的运行状况。
5. 在高并发场景下，考虑使用 `TryEnqueue()` 或 `EnqueueWithRetry()` 来避免阻塞。

## 8. 注意事项

1. Aggregator 不保证严格的顺序处理，如果需要保持顺序，需要在批处理函数中额外实现。
2. 在使用 `Stop()` 或 `SafeStop()` 后，不应再尝试向Aggregator中添加新的数据项。
3. 批处理函数应该是幂等的，因为在某些错误情况下可能会重试处理同一批数据。
4. 注意设置合适的 `LingerTime`，以平衡实时性和批处理效率。

## 9. 结论

Aggregator模块提供了一个高效的方式来批量处理数据，特别适用于需要高吞吐量的场景。通过合理配置和使用，可以显著提高数据处理的效率和可靠性。在使用过程中，需要根据具体的应用场景和需求来调整各项参数，以达到最佳的性能和资源利用。