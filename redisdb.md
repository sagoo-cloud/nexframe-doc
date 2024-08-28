# 基于Redis的数据处理组件

基于Redis实现的基础数据库，方便将redis数据库当成普通数据库使用。它提供了一个灵活的接口来处理Redis操作，支持单机、集群和哨兵模式，并实现了数据插入、查询和监听等功能。

## 2. 主要结构

### 2.1 RedisManager

`RedisManager` 是这个模块的核心结构，它包含以下主要字段：

- `client`: Redis客户端
- `recordDuration`: 记录保持时间
- `recordLimit`: 记录限制数量
- `pipelineBufferSize`: 管道缓冲大小
- `pipelineCounter`: 管道计数器
- `dbname`: 数据库名称

## 3. 初始化和配置

### 3.1 获取RedisManager实例

使用 `DB()` 函数获取 `RedisManager` 的单例实例：

```go
redisManager := redisdb.DB()
```

### 3.2 配置选项

Redis配置通过 `redisOptions` 结构体进行设置，主要包括：

- `Mode`: Redis模式（single/cluster/sentinel）
- `SentinelMasterName`: Sentinel模式下的主节点名称
- `Addr`: Redis服务器地址
- `DB`: Redis数据库编号
- `UserName`: Redis用户名
- `Password`: Redis密码
- `PoolSize`: 连接池大小
- `RecordDuration`: 记录的有效时间
- `RecordLimit`: 记录的条数限制
- `PipelineBufferSize`: 管道缓冲大小

## 4. 主要功能

### 4.1 插入数据

#### 4.1.1 插入单条数据

```go
err := redisManager.InsertData(ctx, key, data, buffer)
```

- `ctx`: 上下文
- `key`: 数据的键
- `data`: 要插入的数据
- `buffer`: 是否使用缓冲处理

#### 4.1.2 批量插入数据

```go
err := redisManager.InsertBatchData(ctx, key, data)
```

- `data`: 要批量插入的数据切片

### 4.2 查询数据

#### 4.2.1 获取最新的数据

```go
data, err := redisManager.GetData(ctx, key)
```

#### 4.2.2 获取最新的一条数据

```go
data, err := redisManager.GetDataByLatest(ctx, key)
```

#### 4.2.3 分页获取数据

```go
data, total, currentPage, err := redisManager.GetDataByPage(ctx, deviceKey, pageNum, pageSize, types, dateRange)
```

- `deviceKey`: 设备键
- `pageNum`: 页码
- `pageSize`: 每页大小
- `types`: 类型过滤
- `dateRange`: 日期范围过滤

### 4.3 监听新数据

```go
redisManager.ListenForNewData(ctx, key, processor, interval)
```

- `processor`: 处理新数据的函数
- `interval`: 轮询间隔

## 5. 错误处理

所有的方法都返回错误，开发者应该妥善处理这些错误。例如：

```go
if err := redisManager.InsertData(ctx, key, data, false); err != nil {
    log.Printf("Failed to insert data: %v", err)
    // 处理错误...
}
```

## 6. 最佳实践

1. 使用单例模式获取 `RedisManager` 实例，避免多次创建连接池。
2. 合理设置 `RecordDuration` 和 `RecordLimit`，避免存储过多无用数据。
3. 在高并发场景下，考虑使用 `InsertBatchData` 和缓冲处理来提高性能。
4. 使用 `ListenForNewData` 进行实时数据处理时，选择合适的轮询间隔。
5. 在应用退出时，确保正确关闭Redis连接。

## 7. 注意事项

1. 这个模块使用了前缀 `deviceCacheData:` 来标识设备数据缓存，使用时需要注意避免键名冲突。
2. 在集群模式下，需要确保所有相关的Redis操作都在同一个槽（slot）中执行。
3. 使用 Sentinel 模式时，需要正确配置主节点名称和哨兵地址。
4. 数据插入使用了 Redis 的列表结构，查询时需要注意数据的顺序。

## 8. 结论

这个Redis数据库管理模块提供了一个强大而灵活的接口来处理设备数据的存储和检索。通过合理使用这些功能，开发人员可以轻松地在应用中集成Redis数据库操作，实现高效的数据管理和实时处理。