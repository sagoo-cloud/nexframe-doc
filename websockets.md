# WebSocket服务

## 1. 概述

这个WebSocket服务器程序是基于Go语言的`gorilla/websocket`包构建的，提供了一个灵活的WebSocket服务器实现。该程序主要包含以下功能：

1. WebSocket连接管理
2. 消息路由和处理
3. 服务器配置选项
4. 并发控制

## 2. 主要结构

### 2.1 Server 结构体

`Server`结构体是这个WebSocket服务器的核心，它包含了以下主要字段：

- `handlers`: 存储消息处理器的映射
- `upgrader`: WebSocket连接升级器
- `maxConns`: 最大连接数
- `activeConns`: 当前活跃连接数
- `ctx`: 服务器上下文
- `logger`: 日志记录器

## 3. 服务器初始化

### 3.1 创建新服务器

使用`NewServer`函数创建一个新的WebSocket服务器实例：

```go
server := websockets.NewServer()
```

### 3.2 服务器配置选项

你可以使用以下选项来配置服务器：

- `WithMaxConnections(n int)`: 设置最大连接数
- `WithLogger(logger *slog.Logger)`: 设置自定义日志记录器

示例：

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
server := websockets.NewServer(
    websockets.WithMaxConnections(2000),
    websockets.WithLogger(logger),
)
```

## 4. 注册消息处理器

使用`Register`方法注册消息处理器：

```go
handler := &commons.CommHandler{
    Handle: func(ctx context.Context, message []byte) ([]byte, error) {
        // 处理消息逻辑
        return []byte("Response"), nil
    },
}
server.Register("route_name", handler)
```

## 5. 启动服务器

使用`Serve`方法启动WebSocket服务器：

```go
err := server.Serve(":8080")
if err != nil {
    log.Fatal("服务器启动失败:", err)
}
```

## 6. 消息处理

### 6.1 消息格式

客户端发送的消息应该是JSON格式，包含以下字段：

- `Route`: 字符串，指定要使用的处理器
- `Params`: 任意JSON对象，作为消息参数

示例：

```json
{
    "Route": "echo",
    "Params": {
        "message": "Hello, WebSocket!"
    }
}
```

### 6.2 响应格式

服务器的响应也是JSON格式，结构取决于处理器的实现。

## 7. 错误处理

服务器会自动处理以下错误情况：

- JSON解析错误
- 未找到指定的消息处理器
- 处理器执行错误

错误响应的格式：

```json
{
    "code": 1,
    "msg": "错误描述",
    "data": null
}
```

## 8. 并发控制

服务器实现了基本的并发控制：

- 限制最大连接数
- 使用互斥锁保护共享资源

## 9. 关闭服务器

使用`Close`方法优雅地关闭服务器：

```go
err := server.Close()
if err != nil {
    log.Println("服务器关闭错误:", err)
}
```

## 10. 最佳实践

1. 始终处理`Serve`方法返回的错误。
2. 实现适当的`CheckOrigin`函数来增强安全性。
3. 根据需求调整最大连接数和超时设置。
4. 使用自定义logger进行更好的日志管理。
5. 在生产环境中实现更复杂的错误处理和恢复机制。

## 11. 注意事项

1. 这个实现使用了简单的JSON消息格式。根据需求，你可能需要实现更复杂的协议。
2. 当前实现允许所有源的WebSocket连接。在生产环境中，你应该实现更严格的源检查。
3. 错误处理和日志记录可能需要根据具体应用进行调整。
4. 这个实现没有包含身份验证机制，如果需要，你应该添加相应的功能。


这个WebSocket服务器程序提供了一个灵活、可扩展的框架来处理WebSocket连接和消息。通过合理使用这些功能，你可以轻松地在你的应用程序中集成WebSocket通信功能。如果遇到任何问题或需要进一步的帮助，请查阅源代码或联系维护者。