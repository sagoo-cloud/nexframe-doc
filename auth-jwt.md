# JWT认证中间件开发指导手册

## 1. 简介

这个JWT（JSON Web Token）认证中间件是为Go语言web应用程序设计的。它提供了一种简单而安全的方式来实现用户认证。本指南将帮助您了解如何在您的项目中集成和使用这个中间件。

## 2. 主要功能

- JWT令牌的生成和解析
- 灵活的令牌提取方式（请求头、URL参数、Cookie）
- 可自定义的错误处理
- 与gorilla/mux兼容的中间件

## 3. 安装

首先，确保您的项目中已经安装了必要的依赖：

```bash
go get -u github.com/golang-jwt/jwt/v5
```

然后，将`jwt.go`文件复制到您项目的`auth`包中。

## 4. 基本使用

### 4.1 创建中间件实例

```go
import (
    "github.com/sagoo-cloud/nexframe/auth"
    "github.com/golang-jwt/jwt/v5"
)

func main() {
    // 创建JWT配置
    jwtConfig := auth.JwtConfig{
        SigningKey:    []byte("your-secret-key"),
        TokenLookup:   "header:Authorization",
        SigningMethod: "HS256",
    }

    // 创建中间件实例
    jwtMiddleware, err := auth.New(jwtConfig)
    if err != nil {
        // 处理错误
    }

    // ... 配置您的路由器
    router.Use(jwtMiddleware.Middleware)
}
```

### 4.2 生成令牌

```go
claims := &auth.TokenClaims{
    Username: "example_user",
    RegisteredClaims: jwt.RegisteredClaims{
        ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
    },
}

token, err := auth.GenerateToken("your-secret-key", auth.WithClaims(func() jwt.Claims { return claims }))
if err != nil {
    // 处理错误
}
```

### 4.3 解析令牌

```go
func someProtectedHandler(w http.ResponseWriter, r *http.Request) {
    claims, ok := auth.FromContext(r.Context())
    if !ok {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }

    // 使用claims中的信息...
    username := claims.(*auth.TokenClaims).Username
    // ...
}
```

## 5. 高级配置

### 5.1 自定义令牌查找方式

您可以自定义从请求中提取令牌的方式：

```go
jwtConfig := auth.JwtConfig{
    // ...
    TokenLookup: "header:X-API-Key",  // 从自定义头中提取
    // 或
    TokenLookup: "query:token",       // 从URL参数中提取
    // 或
    TokenLookup: "cookie:jwt",        // 从Cookie中提取
}
```

### 5.2 自定义错误处理

```go
jwtConfig := auth.JwtConfig{
    // ...
    ErrHandler: func(w http.ResponseWriter, r *http.Request, err error) {
        // 自定义错误响应
        http.Error(w, "Authentication failed: "+err.Error(), http.StatusUnauthorized)
    },
}
```

### 5.3 使用不同的签名方法

```go
jwtConfig := auth.JwtConfig{
    // ...
    SigningMethod: "RS256",
    SigningKey: rsaPublicKey,  // 对于RS256，这里应该是rsa.PublicKey
}

// 生成令牌时
token, err := auth.GenerateToken("", 
    auth.WithSigningMethod(jwt.SigningMethodRS256),
    auth.WithKeyFunc(func(token *jwt.Token) (interface{}, error) {
        return rsaPrivateKey, nil
    }),
)
```

## 6. 最佳实践

1. **安全性**：
    - 使用足够长且复杂的密钥
    - 定期轮换密钥
    - 在生产环境中使用HTTPS

2. **性能**：
    - 合理设置令牌的过期时间
    - 考虑实现令牌刷新机制

3. **错误处理**：
    - 提供有意义的错误消息，但不要泄露敏感信息
    - 记录认证失败的日志，但要注意不要记录敏感数据

4. **声明使用**：
    - 只在令牌中包含必要的信息
    - 对于敏感信息，考虑使用引用而不是直接包含

## 7. 故障排除

1. "缺少认证头"错误：确保客户端正确设置了Authorization头。

2. "无效的认证头"错误：检查令牌格式，确保使用了"Bearer "前缀。

3. 令牌解析失败：检查签名方法和密钥是否匹配。

4. 令牌过期：检查令牌的过期时间设置，考虑实现刷新机制。

## 8. 结语

这个JWT中间件提供了一个灵活且安全的方式来为您的Go web应用程序添加认证功能。通过遵循本指南，您应该能够轻松地集成和使用这个中间件。如果遇到任何问题，请查阅源代码或寻求进一步的帮助。

记住，安全性是一个持续的过程。定期审查和更新您的认证策略是很重要的。