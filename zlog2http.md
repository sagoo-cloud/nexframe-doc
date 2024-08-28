# 通过HTTP发送日志系统

## 目录
1. 简介
2. 设置LogHTTPWriter
3. 与gorilla/mux和zerolog集成
4. 设置日志接收服务器
5. 测试日志系统
6. 最佳实践和提示
7. 故障排除

## 1. 简介

本手册描述了如何设置和使用我们的HTTP日志系统，包括：
- 用于通过HTTP发送日志的自定义`LogHTTPWriter`
- 与gorilla/mux（用于路由）和zerolog（用于日志生成）的集成
- 用于测试目的的日志接收服务器

## 2. 设置LogHTTPWriter

### 2.1 LogHTTPWriter结构

`LogHTTPWriter`的定义如下：

```go
type LogHTTPWriter struct {
    Writer io.Writer
    serverURL     string
    warnOnHttpErr bool
    errorLogger   zerolog.Logger
}
```

### 2.2 创建新的LogHTTPWriter

使用`NewLogHTTPWriter`函数创建新实例：

```go
func NewLogHTTPWriter(serverURL string, warnOnHttpErr bool) *LogHTTPWriter
```

使用示例：
```go
httpWriter := zlog.NewLogHTTPWriter("http://your-log-server.com/logs", true)
```

### 2.3 Write方法

`Write`方法将日志数据发送到指定的服务器：

```go
func (w *LogHTTPWriter) Write(p []byte) (n int, err error)
```

当您将此writer与zerolog一起使用时，此方法会自动调用。

## 3. 与gorilla/mux和zerolog集成

### 3.1 设置Logger

创建一个多重写入器，同时写入控制台和HTTP：

```go
multiWriter := zerolog.MultiLevelWriter(zerolog.ConsoleWriter{Out: os.Stdout}, httpWriter)
logger := zerolog.New(multiWriter).With().Timestamp().Logger()
```

### 3.2 创建日志中间件

定义一个中间件来记录所有传入的请求：

```go
loggingMiddleware := func(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        logger.Info().
            Str("method", r.Method).
            Str("url", r.URL.String()).
            Msg("收到请求")
        next.ServeHTTP(w, r)
    })
}
```

### 3.3 设置mux路由器

创建mux路由器并应用中间件：

```go
r := mux.NewRouter()
r.Use(loggingMiddleware)
```

### 3.4 定义路由

将您的路由添加到mux路由器：

```go
r.HandleFunc("/", HomeHandler)
r.HandleFunc("/api", APIHandler)
```

### 3.5 启动服务器

使用mux路由器启动您的HTTP服务器：

```go
http.ListenAndServe(":8080", r)
```

## 4. 设置日志接收服务器

### 4.1 日志接收服务器代码

将以下代码保存为`log_receiver.go`：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "time"
)

func main() {
    http.HandleFunc("/logs", handleLogs)
    
    port := "8080"
    fmt.Printf("正在启动日志接收服务器，端口 %s...\n", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}

func handleLogs(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "只允许POST方法", http.StatusMethodNotAllowed)
        return
    }

    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "读取请求体时出错", http.StatusInternalServerError)
        return
    }
    defer r.Body.Close()

    fmt.Printf("在 %s 收到日志：\n%s\n", time.Now().Format(time.RFC3339), string(body))

    filename := "received_logs.txt"
    f, err := os.OpenFile(filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
    if err != nil {
        log.Printf("打开文件时出错：%v", err)
    } else {
        defer f.Close()
        if _, err := f.WriteString(fmt.Sprintf("--- 在 %s 收到日志 ---\n%s\n\n", time.Now().Format(time.RFC3339), string(body))); err != nil {
            log.Printf("写入文件时出错：%v", err)
        }
    }

    w.WriteHeader(http.StatusOK)
    w.Write([]byte("成功接收日志"))
}
```

### 4.2 运行日志接收服务器

使用以下命令运行服务器：

```
go run log_receiver.go
```

服务器将在8080端口启动。

## 5. 测试日志系统

### 5.1 配置LogHTTPWriter

更新您的`LogHTTPWriter`以指向本地接收器：

```go
httpWriter := zlog.NewLogHTTPWriter("http://localhost:8080/logs", true)
```

### 5.2 运行您的应用程序

启动使用日志系统的主应用程序。

### 5.3 生成日志

与您的应用程序交互以生成日志。这些日志应该被发送到日志接收服务器。

### 5.4 检查接收到的日志

- 在运行日志接收服务器的控制台中查看日志。
- 检查`received_logs.txt`文件以查看所有接收到的日志记录。

## 6. 最佳实践和提示

- 使用zerolog的结构化日志功能，以便于解析和分析。
- 在日志中包含相关上下文（例如，请求ID，用户ID）。
- 使用适当的日志级别（Info、Warn、Error等）来分类您的日志。
- 定期轮换和归档日志文件以管理磁盘空间。
- 在生产环境中，考虑使用更强大的日志聚合系统。

## 7. 故障排除

- 如果日志未出现在接收器中，请检查网络连接和防火墙设置。
- 确保`LogHTTPWriter`中的`serverURL`与您的日志接收服务器地址匹配。
- 如果您看到有关HTTP错误的警告，请检查您的日志接收服务器是否正在运行且可访问。
- 对于zerolog或mux的任何问题，请查阅它们各自的文档。

请记住，随着您对日志系统进行更改或改进，更新此手册。祝您开发顺利！