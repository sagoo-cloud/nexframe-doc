# Gorilla Mux Session 中间件开发手册

## 1. 简介

这个 Session 中间件用于在 Web 应用中轻松管理用户会话。它提供了一种简单的方式来设置、获取和管理会话数据。

## 2. 安装

首先，确保您已经安装了必要的依赖：

```bash
go get -u github.com/gorilla/sessions
go get -u github.com/gorilla/context
```

然后，将 session 包添加到您的项目中。

## 3. 基本用法

### 3.1 创建和使用中间件

最简单的使用方式是使用默认配置：

```go
package main

import (
    "net/http"
    "github.com/sagoo-cloud/nexframe/session"
    "github.com/gorilla/mux"
    "github.com/gorilla/sessions"
)

func main() {
   
	srv := nexframe.Server()

	// 创建一个 cookie 存储
    store := sessions.NewCookieStore([]byte("your-secret-key"))
    
    // 使用 session 中间件
	srv.WithMiddleware(session.Middleware(store))

  
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    // 获取会话
    sess, err := session.Get("my-session", r)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // 使用会话
    sess.Values["visits"] = sess.Values["visits"].(int) + 1
    sess.Save(r, w)

    w.Write([]byte("Welcome! Visit count: " + fmt.Sprintf("%d", sess.Values["visits"])))
}
```

### 3.2 使用自定义配置

如果您需要更多控制，可以使用自定义配置：

```go
config := session.Config{
    Skipper: func(r *http.Request) bool {
        return r.URL.Path == "/public"
    },
    Store: sessions.NewCookieStore([]byte("your-secret-key")),
}

srv.WithMiddleware(session.MiddlewareWithConfig(config))
```

## 4. 配置选项

### 4.1 Config 结构体

```go
type Config struct {
    Skipper func(r *http.Request) bool
    Store   sessions.Store
}
```

- `Skipper`: 一个函数，用于决定是否跳过某些请求的会话处理。
- `Store`: 会话存储接口，用于保存会话数据。

### 4.2 默认配置

```go
var DefaultConfig = Config{
    Skipper: defaultSkipper,
}

func defaultSkipper(r *http.Request) bool {
    return false
}
```

默认配置使用 `defaultSkipper`，它会处理所有请求（不跳过任何请求）。

## 5. 核心功能

### 5.1 获取会话

使用 `Get` 函数来获取指定名称的会话：

```go
sess, err := session.Get("session-name", r)
if err != nil {
    // 处理错误
}
```

### 5.2 使用会话数据

会话数据存储在 `sess.Values` 中，它是一个 `map[interface{}]interface{}`：

```go
// 设置值
sess.Values["user_id"] = 12345

// 获取值
if userID, ok := sess.Values["user_id"].(int); ok {
    fmt.Printf("User ID: %d\n", userID)
}

// 删除值
delete(sess.Values, "user_id")

// 保存更改
sess.Save(r, w)
```

## 6. 高级用法

### 6.1 自定义存储

您可以实现自己的 `sessions.Store` 接口来使用自定义存储：

```go
type MyStore struct {
    // 自定义字段
}

func (s *MyStore) Get(r *http.Request, name string) (*sessions.Session, error) {
    // 实现获取会话的逻辑
}

func (s *MyStore) New(r *http.Request, name string) (*sessions.Session, error) {
    // 实现创建新会话的逻辑
}

func (s *MyStore) Save(r *http.Request, w http.ResponseWriter, sess *sessions.Session) error {
    // 实现保存会话的逻辑
}

// 使用自定义存储
myStore := &MyStore{}
r.Use(session.Middleware(myStore))
```

### 6.2 会话过期

Gorilla Sessions 包支持会话过期。您可以在创建会话时设置 `MaxAge` 选项：

```go
sess, _ := session.Get("my-session", r)
sess.Options = &sessions.Options{
    MaxAge:   86400 * 7, // 7 天
    HttpOnly: true,
    Secure:   true,  // 仅在 HTTPS 时使用
}
sess.Save(r, w)
```

## 7. 最佳实践

1. 始终使用强密钥来创建会话存储。
2. 在生产环境中使用 HTTPS，并设置 `Secure: true`。
3. 对敏感数据进行加密后再存储在会话中。
4. 定期轮换会话密钥。
5. 实现会话注销功能，清除服务器端和客户端的会话数据。

## 8. 故障排除

1. 如果无法获取会话，检查是否正确设置了中间件。
2. 确保在修改会话后调用 `sess.Save(r, w)`。
3. 如果会话数据未持久化，检查存储配置是否正确。
