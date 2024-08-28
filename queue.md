# 消息队列服务

## 1. 简介

本消息队列系统提供了一个统一的接口，支持 Redis 和 RocketMQ 两种底层实现。它允许开发者在不同的消息队列系统之间无缝切换，而无需修改业务逻辑代码。

## 2. 安装

首先，确保您的项目中已经包含了必要的依赖：

```bash
go get github.com/sagoo-cloud/nexframe/queue
go get github.com/sagoo-cloud/nexframe/queue/redisqueue
go get github.com/sagoo-cloud/nexframe/queue/rocketqueue
```

## 3. 配置

### 3.1 Redis 配置

确保您的项目中有正确的 Redis 连接配置。通常，这可能在一个配置文件或环境变量中设置。

### 3.2 RocketMQ 配置

对于 RocketMQ，您需要设置 NameServer 地址和生产者/消费者组名。这些通常在配置文件中设置。

## 4. 使用方法

### 4.1 获取队列实例

```go
import (
    "github.com/sagoo-cloud/nexframe/queue"
    "github.com/sagoo-cloud/nexframe/queue/redisqueue"
    "github.com/sagoo-cloud/nexframe/queue/rocketqueue"
)

// 获取 Redis 队列实例
redisQueue := queue.GetQueue("myRedisQueue", queue.DriverTypeRedis)

// 获取 RocketMQ 队列实例
rocketQueue := queue.GetQueue("myRocketQueue", queue.DriverTypeRocketMq)
```

### 4.2 消息入队

```go
ctx := context.Background()
key := "myQueueKey"
message := "Hello, World!"

ok, err := redisQueue.Enqueue(ctx, key, message)
if err != nil {
    log.Printf("入队失败: %v", err)
    return
}
if !ok {
    log.Println("入队操作未成功执行")
    return
}
log.Println("消息已成功入队")
```

### 4.3 消息出队

```go
message, tag, token, dequeueCount, err := redisQueue.Dequeue(ctx, key)
if err != nil {
    log.Printf("出队失败: %v", err)
    return
}
log.Printf("收到消息: %s, 标签: %s, 令牌: %s, 出队次数: %d", message, tag, token, dequeueCount)
```

### 4.4 确认消息

```go
ok, err := redisQueue.AckMsg(ctx, key, token)
if err != nil {
    log.Printf("确认消息失败: %v", err)
    return
}
if !ok {
    log.Println("消息确认操作未成功执行")
    return
}
log.Println("消息已成功确认")
```

### 4.5 批量入队

```go
messages := []string{"消息1", "消息2", "消息3"}
ok, err := redisQueue.BatchEnqueue(ctx, key, messages)
if err != nil {
    log.Printf("批量入队失败: %v", err)
    return
}
if !ok {
    log.Println("批量入队操作未成功执行")
    return
}
log.Println("消息已成功批量入队")
```

## 5. 高级用法

### 5.1 使用 RocketMQ 的消息标签

RocketMQ 支持消息标签，可以在入队时设置：

```go
msg := &primitive.Message{
    Topic: key,
    Body:  []byte(message),
}
msg.WithTag("important")

res, err := rocketQueue.(*rocketqueue.RocketQueue).Client.Producer.SendSync(ctx, msg)
if err != nil {
    log.Printf("带标签的消息入队失败: %v", err)
    return
}
log.Printf("带标签的消息已成功入队，消息ID: %s", res.MsgID)
```

### 5.2 处理重试和死信队列

对于 RocketMQ，您可能需要处理消息重试和死信队列的情况：

```go
func consumeMessage(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
    for _, msg := range msgs {
        if msg.ReconsumeTimes > 3 {
            // 消息重试次数过多，可能需要移到死信队列
            log.Printf("消息 %s 重试次数过多，考虑移到死信队列", msg.MsgId)
            return consumer.ConsumeRetryLater, nil
        }
        
        // 处理消息...
        
        if /* 处理成功 */ {
            return consumer.ConsumeSuccess, nil
        } else {
            // 处理失败，稍后重试
            return consumer.ConsumeRetryLater, nil
        }
    }
    return consumer.ConsumeSuccess, nil
}
```

## 6. 最佳实践

1. **错误处理**：始终检查返回的错误，并适当地处理它们。

2. **上下文使用**：使用 `context.Context` 来控制超时和取消操作。

3. **优雅关闭**：在应用程序关闭时，确保正确关闭队列连接：

```go
func gracefulShutdown(q queue.Queue) {
    if rq, ok := q.(*rocketqueue.RocketQueue); ok {
        rq.Shutdown()
    }
    // 对于 Redis，通常不需要特殊的关闭操作
}
```

4. **监控和日志**：实现适当的监控和日志记录，以便及时发现和解决问题。

5. **批量操作**：当需要处理大量消息时，优先使用批量操作以提高效率。

6. **消息持久化**：对于重要的消息，确保使用持久化设置，特别是在使用 Redis 时。

## 7. 故障排除

1. 如果消息无法入队，检查队列连接和权限设置。
2. 对于 RocketMQ，确保 NameServer 地址正确，并且生产者/消费者组名称正确配置。
3. 如果消息丢失，检查是否正确地确认了消息（对于需要显式确认的队列类型）。
4. 对于性能问题，考虑增加连接池大小或使用批量操作。

## 8. 示例应用

以下是一个简单的生产者-消费者示例，展示了如何在实际应用中使用这个消息队列系统：

```go
package main

import (
    "context"
    "log"
    "time"

    "github.com/sagoo-cloud/nexframe/queue"
    _ "github.com/sagoo-cloud/nexframe/queue/redisqueue"
)

func main() {
    // 获取 Redis 队列实例
    redisQueue := queue.GetQueue("myRedisQueue", queue.DriverTypeRedis)

    // 启动生产者
    go producer(redisQueue)

    // 启动消费者
    go consumer(redisQueue)

    // 运行一段时间后退出
    time.Sleep(1 * time.Minute)
}

func producer(q queue.Queue) {
    ctx := context.Background()
    key := "myQueue"

    for i := 0; i < 10; i++ {
        message := fmt.Sprintf("消息 #%d", i)
        ok, err := q.Enqueue(ctx, key, message)
        if err != nil {
            log.Printf("生产者: 入队失败: %v", err)
            continue
        }
        if !ok {
            log.Printf("生产者: 消息 '%s' 入队操作未成功执行", message)
            continue
        }
        log.Printf("生产者: 消息 '%s' 已成功入队", message)
        time.Sleep(1 * time.Second)
    }
}

func consumer(q queue.Queue) {
    ctx := context.Background()
    key := "myQueue"

    for {
        message, _, token, dequeueCount, err := q.Dequeue(ctx, key)
        if err != nil {
            log.Printf("消费者: 出队失败: %v", err)
            time.Sleep(1 * time.Second)
            continue
        }
        if message == "" {
            log.Println("消费者: 队列为空，等待新消息...")
            time.Sleep(1 * time.Second)
            continue
        }

        log.Printf("消费者: 收到消息: %s, 出队次数: %d", message, dequeueCount)

        // 处理消息...

        ok, err := q.AckMsg(ctx, key, token)
        if err != nil {
            log.Printf("消费者: 确认消息失败: %v", err)
            continue
        }
        if !ok {
            log.Println("消费者: 消息确认操作未成功执行")
            continue
        }
        log.Println("消费者: 消息已成功确认")
    }
}
```

这个示例展示了如何创建一个简单的生产者-消费者系统，使用 Redis 作为底层队列。生产者持续产生消息并将其加入队列，而消费者则不断地从队列中取出消息并处理它们。

## 

这个消息队列系统提供了一个灵活且易用的接口，支持多种底层实现。通过遵循本手册中的指导和最佳实践，您应该能够在您的应用程序中有效地集成和使用这个消息队列系统。