# MQTT客户端开发手册



## 1. 简介

这是一个基于 Paho MQTT 的客户端封装库，提供了以下特性：
- 自动重连机制
- 可配置的日志系统
- TLS加密支持
- 并发安全
- 优雅关闭
- 上下文控制

## 2. 快速开始

### 2.1 基本使用
```go
import (
    "context"
    "github.com/sagoo-cloud/nexframe/net/mqttclient"
)

func main() {
    // 创建基本配置
    conf := mqttclient.Config{
        Server:   "tcp://localhost:1883",
        Username: "user",
        Password: "password",
    }

    // 创建客户端
    client, err := mqttclient.NewClient(context.Background(), conf)
    if err != nil {
        panic(err)
    }
    defer client.Close()

    // 注册消息处理器
    handler := mqttclient.Handler{
        Topic: "test/topic",
        Qos:   1,
        Handle: func(c paho.Client, msg paho.Message) {
            fmt.Printf("收到消息: %s\n", string(msg.Payload()))
        },
    }

    if err := client.RegisterHandler(handler); err != nil {
        panic(err)
    }

    // 发布消息
    if err := client.Publish("test/topic", 1, []byte("test message")); err != nil {
        panic(err)
    }
}
```

## 3. 配置说明

### 3.1 配置选项
```go
type Config struct {
    Server               string        // MQTT broker地址
    Username             string        // 用户名
    Password             string        // 密码
    MaxReconnectInterval time.Duration // 重连间隔
    QOS                  uint8         // 服务质量等级（0,1,2）
    CleanSession         bool          // 清理会话标志
    ClientID             string        // 客户端ID（为空则自动生成）
    CAFile               string        // CA证书文件路径
    CertFile             string        // 客户端证书文件路径
    CertKeyFile          string        // 客户端密钥文件路径
    Logger               Logger        // 日志记录器
    LogLevel             LogLevel      // 日志级别
    QueueSize            int          // 消息队列大小（默认100）
}
```

### 3.2 TLS配置示例
```go
conf := mqttclient.Config{
    Server:      "ssl://localhost:8883",
    CAFile:      "/path/to/ca.crt",
    CertFile:    "/path/to/client.crt",
    CertKeyFile: "/path/to/client.key",
}
```

## 4. API参考

### 4.1 客户端创建和关闭
```go
// 创建客户端
func NewClient(ctx context.Context, conf Config) (*Client, error)

// 关闭客户端
func (c *Client) Close() error

// 检查连接状态
func (c *Client) IsConnected() bool
```

### 4.2 消息订阅和发布
```go
// 注册消息处理器
func (c *Client) RegisterHandler(handler Handler) error

// 注销消息处理器
func (c *Client) UnregisterHandler(topic string) error

// 获取处理器
func (c *Client) GetHandlerByTopic(topic string) (Handler, bool)

// 发布消息
func (c *Client) Publish(topic string, qos byte, data []byte) error
```

### 4.3 日志控制
```go
// 设置日志记录器
func (c *Client) SetLogger(logger Logger)

// 设置日志级别
func (c *Client) SetLogLevel(level LogLevel)
```

## 5. 日志管理

### 5.1 日志级别
```go
const (
    LogLevelNone  LogLevel = iota  // 不输出日志
    LogLevelError                  // 只输出错误日志
    LogLevelWarn                   // 输出警告和错误日志
    LogLevelInfo                   // 输出信息、警告和错误日志
    LogLevelDebug                  // 输出所有日志
)
```

### 5.2 自定义日志实现
```go
type CustomLogger struct {
    // 自定义字段
}

// 实现Logger接口
func (l *CustomLogger) Debug(format string, v ...interface{}) {
    // 实现Debug日志
}

func (l *CustomLogger) Info(format string, v ...interface{}) {
    // 实现Info日志
}

func (l *CustomLogger) Warn(format string, v ...interface{}) {
    // 实现Warn日志
}

func (l *CustomLogger) Error(format string, v ...interface{}) {
    // 实现Error日志
}

// 使用自定义日志器
conf := mqttclient.Config{
    Logger:   &CustomLogger{},
    LogLevel: mqttclient.LogLevelInfo,
}
```

## 6. 错误处理

### 6.1 错误类型
```go
var (
    ErrNilHandler         // 处理器为空
    ErrClientClosed       // 客户端已关闭
    ErrInvalidConfig      // 配置无效
    ErrSubscriptionFailed // 订阅失败
    ErrConnectionFailed   // 连接失败
    ErrPublishFailed      // 发布失败
    ErrUnsubscribeFailed  // 取消订阅失败
)
```

### 6.2 错误处理示例
```go
if err := client.RegisterHandler(handler); err != nil {
    switch {
    case errors.Is(err, mqttclient.ErrNilHandler):
        // 处理空处理器错误
    case errors.Is(err, mqttclient.ErrClientClosed):
        // 处理客户端关闭错误
    case errors.Is(err, mqttclient.ErrSubscriptionFailed):
        // 处理订阅失败错误
    default:
        // 处理其他错误
    }
}
```

## 7. 最佳实践

### 7.1 连接管理
```go
// 使用context控制连接超时
ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
defer cancel()

client, err := mqttclient.NewClient(ctx, conf)
if err != nil {
    // 处理错误
}
```

### 7.2 消息处理
```go
// 使用QoS 1确保消息可靠性
handler := mqttclient.Handler{
    Topic: "important/topic",
    Qos:   1,
    Handle: func(c paho.Client, msg paho.Message) {
        // 1. 首先验证消息
        if !validateMessage(msg.Payload()) {
            return
        }

        // 2. 处理消息
        if err := processMessage(msg.Payload()); err != nil {
            // 3. 错误处理
            handleError(err)
            return
        }

        // 4. 确认处理完成
        confirmProcessing(msg.Topic())
    },
}
```

### 7.3 资源管理
```go
// 确保资源正确释放
client, err := mqttclient.NewClient(ctx, conf)
if err != nil {
    return err
}
defer func() {
    if err := client.Close(); err != nil {
        log.Printf("关闭客户端出错: %v", err)
    }
}()
```

## 8. 常见问题

### Q1: 如何处理断线重连？
A1: 客户端已内置自动重连机制，可通过 MaxReconnectInterval 配置重连间隔。建议在关键业务场景下监控连接状态：
```go
go func() {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    for range ticker.C {
        if !client.IsConnected() {
            log.Println("连接已断开，等待自动重连")
        }
    }
}()
```

### Q2: 如何确保消息不丢失？
A2:
1. 使用 QoS 1 或 2 级别
2. 实现消息确认机制
3. 适当设置 QueueSize

### Q3: 订阅主题的最佳实践？
A3:
1. 使用具体的主题而不是通配符（如果可能）
2. 避免订阅 '#' 这样的广泛主题
3. 为不同类型的消息使用不同的主题

### Q4: 性能优化建议？
A4:
1. 适当设置 QueueSize
2. 使用合适的 QoS 级别
3. 避免在消息处理器中执行耗时操作
4. 考虑使用消息批处理

如需更多帮助，请参考示例代码或提交Issue。