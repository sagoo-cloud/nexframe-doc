# MQTT订阅服务

## 1. 概述

这个MQTT订阅服务的MQTT客户端库是基于 `github.com/eclipse/paho.mqtt.golang` 包构建的，提供了一个简单的接口来实现MQTT的连接、发布和订阅功能。该库主要包含三个主要组件：

1. MQTT客户端实例管理（ins.go）
2. 消息发布功能（publish.go）
3. 消息订阅服务器（subscribe_server.go）

## 2. MQTT客户端实例管理 (ins.go)

### 2.1 功能描述

`ins.go` 文件负责管理MQTT客户端实例。它使用单例模式确保整个应用程序中只有一个MQTT客户端实例。

### 2.2 主要函数

- `GetIns()`: 获取MQTT客户端实例
- `init_mc()`: 初始化MQTT客户端

### 2.3 使用示例

```go
import "github.com/sagoo-cloud/nexframe/servers/mqtts"

// 获取MQTT客户端实例
client := mqtts.GetIns()

// 使用客户端进行操作
if client != nil {
    // 执行MQTT操作
} else {
    // 处理连接失败的情况
}
```

## 3. 消息发布功能 (publish.go)

### 3.1 功能描述

`publish.go` 文件提供了一个简单的接口来发布MQTT消息。

### 3.2 主要函数

- `Publish(topic string, payload interface{}) error`: 发布消息到指定的主题

### 3.3 使用示例

```go
import "github.com/sagoo-cloud/nexframe/servers/mqtts"

// 定义要发布的数据
data := map[string]string{
    "message": "Hello, MQTT!",
}

// 发布消息
err := mqtts.Publish("your/topic", data)
if err != nil {
    // 处理发布错误
    log.Printf("发布失败: %v", err)
} else {
    log.Println("消息发布成功")
}
```

## 4. 消息订阅服务器 (subscribe_server.go)

### 4.1 功能描述

`subscribe_server.go` 文件实现了一个MQTT订阅服务器，可以处理多个主题的订阅。

### 4.2 主要结构和函数

- `Server` 结构体：订阅服务器的主要结构
- `NewServer()`: 创建新的订阅服务器
- `Register(name string, handler *commons.CommHandler)`: 注册主题和对应的处理器
- `Serve()`: 启动订阅服务
- `Close()`: 关闭订阅服务

### 4.3 使用示例

```go
import (
    "github.com/sagoo-cloud/nexframe/servers/mqtts/mqtts"
    "github.com/sagoo-cloud/nexframe/servers/commons"
    "context"
)

// 创建消息处理器
handler := &commons.CommHandler{
    Handle: func(ctx context.Context, message []byte) ([]byte, error) {
        // 处理接收到的消息
        log.Printf("收到消息: %s", string(message))
        return []byte("处理完成"), nil
    },
}

// 创建订阅服务器
server := mqtts.NewServer()

// 注册主题和处理器
server.Register("your/topic", handler)

// 启动服务
go func() {
    if err := server.Serve(); err != nil {
        log.Printf("服务器错误: %v", err)
    }
}()

// ... 主程序逻辑 ...

// 在程序结束时关闭服务
defer server.Close()
```

## 5. 配置

这个库使用 `configs.LoadMqttConfig()` 来加载MQTT配置。确保在你的配置文件中包含以下MQTT相关的设置：

- Host: MQTT broker的地址
- UserName: MQTT认证用户名
- PassWord: MQTT认证密码
- ClientID: 客户端ID
- PublishQos: 发布消息的QoS级别
- SubscribeQos: 订阅消息的QoS级别
- Parallel: 是否并行处理订阅消息

## 6. 注意事项

1. 确保在使用任何功能之前，MQTT客户端已成功连接。
2. 处理所有可能的错误，特别是在发布消息和启动订阅服务时。
3. 在应用程序退出时，记得调用 `Close()` 函数来清理订阅和断开连接。
4. 根据你的需求调整 QoS 级别和并行处理选项。


这个MQTT客户端库提供了一个简洁的接口来处理MQTT通信。通过合理使用这些功能，你可以轻松地在你的应用程序中集成MQTT消息传递功能。如果遇到任何问题或需要进一步的帮助，请查阅源代码或联系库的维护者。