# JWT 中间件使用手册

## 1. 简介

JWT（JSON Web Token）中间件是一个用于 Go 语言的认证中间件，专门设计用于与 gorilla/mux 路由器配合使用。它提供了一种简单而安全的方法来实现基于令牌的身份验证。

主要特性：
- 支持从不同来源提取令牌（头部、查询参数、cookie）
- 可自定义令牌验证逻辑
- 灵活的错误处理
- 易于与现有 gorilla/mux 应用程序集成

## 2. 安装

要使用 JWT 中间件，首先需要安装它。在您的项目目录中运行以下命令：

```bash
go get github.com/sagoo-cloud/nexframe
```

然后在您的 Go 文件中导入它：

```go
import "github.com/sagoo-cloud/nexframe/auth"

```

## 3. 基本使用

以下是一个基本的使用示例：

```go
package main

import (
    "github.com/gorilla/mux"
    "github.com/sagoo-cloud/nexframe/auth"
    "net/http"
)

var secretKey = []byte("your-secret-key")

func main() {
    r := mux.NewRouter()

    // 创建 JWT 中间件
    jwtMiddleware, err := auth.New(auth.JwtConfig{
        SigningKey: secretKey,
    })
    if err != nil {
        panic(err)
    }

    // 应用中间件到需要保护的路由
    api := r.PathPrefix("/api").Subrouter()
    api.Use(jwtMiddleware.Middleware)

    // 设置路由
    api.HandleFunc("/protected", protectedHandler).Methods("GET")
    r.HandleFunc("/login", loginHandler).Methods("POST")

    http.ListenAndServe(":8080", r)
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
    // 从上下文中获取令牌声明
    claims := r.Context().Value("user").(*auth.TokenClaims)
    w.Write([]byte("Welcome, " + claims.Username))
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
    // 这里应该有用户认证逻辑
    claims := auth.TokenClaims{
        Username: "example_user",
    }
    auth.CreateAndSendToken(w, claims, secretKey, 24*time.Hour)
}
```

## 4. 配置选项

JWT 中间件提供了多个配置选项，可以在创建中间件实例时设置：

```go
jwtMiddleware, err := auth.New(auth.JwtConfig{
    SigningKey:    []byte("your-secret-key"),
    SigningMethod: "HS256",
    TokenLookup:   "header:Authorization",
    ContextKey:    "user",
    ErrorHandler:  customErrorHandler,
})
```

- `SigningKey`：用于签名和验证令牌的密钥（必需）
- `SigningMethod`：令牌签名方法（默认为 "HS256"）
- `TokenLookup`：定义如何查找令牌（默认为 "header:Authorization"）
- `ContextKey`：用于在请求上下文中存储令牌声明的键（默认为 "user"）
- `ErrorHandler`：自定义错误处理函数

## 5. 高级用法

### 5.1 自定义令牌提取

您可以自定义从请求中提取令牌的方式：

```go
jwtMiddleware, _ := auth.New(auth.JwtConfig{
    SigningKey:  secretKey,
    TokenLookup: "cookie:jwt",
})
```

这将从名为 "jwt" 的 cookie 中提取令牌。

### 5.2 自定义声明

您可以扩展 `TokenClaims` 结构体以包含额外的自定义字段：

```go
type CustomClaims struct {
    auth.TokenClaims
    UserID int `json:"user_id"`
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
    claims := CustomClaims{
        TokenClaims: auth.TokenClaims{Username: "example_user"},
        UserID:      123,
    }
    auth.CreateAndSendToken(w, claims, secretKey, 24*time.Hour)
}
```

## 6. 错误处理

您可以提供自定义的错误处理函数：

```go
func customErrorHandler(w http.ResponseWriter, r *http.Request, err error) {
    http.Error(w, "认证失败: "+err.Error(), http.StatusUnauthorized)
}

jwtMiddleware, _ := auth.New(auth.JwtConfig{
    SigningKey:   secretKey,
    ErrorHandler: customErrorHandler,
})
```

## 7. 最佳实践

1. 始终使用强密钥并保持其安全性。
2. 在生产环境中使用 HTTPS 来保护令牌传输。
3. 设置合理的令牌过期时间。
4. 定期轮换密钥。
5. 在令牌中只包含必要的信息。

## 8. 常见问题解答

Q: 如何处理令牌过期？
A: 令牌过期会自动由中间件处理。您可以在创建令牌时设置过期时间：
```go
auth.CreateAndSendToken(w, claims, secretKey, 1*time.Hour) // 令牌将在1小时后过期
```

Q: 如何在受保护的路由中访问令牌中的信息？
A: 使用 `r.Context().Value("user")` 来获取令牌声明：
```go
func protectedHandler(w http.ResponseWriter, r *http.Request) {
    claims := r.Context().Value("user").(*auth.TokenClaims)
    // 使用 claims.Username 等
}
```

Q: 可以同时从多个来源提取令牌吗？
A: 目前不支持同时从多个来源提取令牌。但您可以根据需要修改 `TokenLookup` 配置。

---

如果您有任何其他问题或需要进一步的帮助，请随时查阅源代码或联系维护者。