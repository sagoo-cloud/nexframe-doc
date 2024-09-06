# 自定义状态码处理

为什么要自定义状态码处理？

默认的HTTP状态码响应通常很简单，缺乏上下文信息。通过自定义状态码处理，我们可以：

1. 提供更详细、更有用的错误信息
2. 统一API的错误响应格式
3. 根据不同的错误状况返回特定的指导或解决方案
4. 增强API的可调试性和用户友好性

我们可以对WebServer指定的状态码进行自定义处理，例如针对常见的404/403/500等错误，我们可以展示自定义的错误信息、页面内容，或者跳转到一个特定的页面。
nexframe 的webserver提供了两个方法来设置自定义的 HTTP 状态码处理函数：`BindStatusHandler` 和 `BindStatusHandlerByMap`。这些方法允许您为特定的 HTTP 状态码定义自定义的响应行为。

## 方法描述

### 1. BindStatusHandler

这个方法允许您为单个 HTTP 状态码绑定一个自定义的处理函数。

#### 语法

```go
BindStatusHandler(status int, handler http.HandlerFunc)
```

#### 参数

- `status`: 一个整数，表示 HTTP 状态码。
- `handler`: 一个 `http.HandlerFunc` 类型的函数，用于处理指定状态码的请求。

#### 使用示例

```go
server.BindStatusHandler(404, func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("自定义 404 - 未找到页面"))
})
```

### 2. BindStatusHandlerByMap

这个方法允许您一次性为多个 HTTP 状态码绑定自定义处理函数。

#### 语法

```go
 BindStatusHandlerByMap(handlers map[int]http.HandlerFunc)
```

#### 参数

- `handlers`: 一个 map，键为整数类型的 HTTP 状态码，值为对应的处理函数。

#### 使用示例

```go
server.BindStatusHandlerByMap(map[int]http.HandlerFunc{
    403: func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("自定义 403 - 禁止访问"))
    },
    404: func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("自定义 404 - 未找到页面"))
    },
    500: func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("自定义 500 - 服务器内部错误"))
    },
})
```

## 注意事项

1. 这些方法会为所有匹配指定状态码的请求设置处理函数，不论请求的 URL 路径是什么。

2. 如果您需要更精细的控制，可能需要在处理函数中添加额外的逻辑。

3. 确保在调用这些方法之前已经正确初始化了 APIFramework 实例。

4. 自定义状态码处理函数会覆盖默认的错误处理行为，请确保您的自定义处理函数提供了足够的信息和适当的响应。

## 最佳实践

- 使用 `BindStatusHandler` 来处理特定的、不常见的状态码。
- 使用 `BindStatusHandlerByMap` 来一次性设置多个常见的错误状态码处理函数，如 400、403、404、500 等。
- 在自定义处理函数中提供清晰、有用的错误信息，以帮助客户端理解并可能解决问题。
- 考虑在处理函数中加入日志记录，以便于调试和监控。

通过使用这些方法，您可以为您的 API 提供更好的错误处理和用户体验。