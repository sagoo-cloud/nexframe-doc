# JWT中间件

JWT（JSON Web Token）中间件包提供了一种简单而安全的方式来处理用户认证。这个包允许你轻松地在你的Go应用程序中集成JWT认证，包括令牌的生成、验证和解析。


## 快速开始

以下是一个基本的使用示例：

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/sagoo-cloud/nexframe/auth"
    "github.com/gorilla/mux"
)

func main() {
    // 创建JWT中间件实例
    jwtMiddleware, err := auth.NewJwt()
    if err != nil {
        panic(err)
    }

    // 创建路由
    r := mux.NewRouter()

    // 应用JWT中间件到受保护的路由
    r.Handle("/protected", jwtMiddleware.Middleware(http.HandlerFunc(protectedHandler))).Methods("GET")

    // 启动服务器
    http.ListenAndServe(":8080", r)
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
    username, err := auth.GetCurrentUser(r.Context())
    if err != nil {
        http.Error(w, "未授权", http.StatusUnauthorized)
        return
    }
    fmt.Fprintf(w, "欢迎, %s!", username)
}
```

## 配置

JWT中间件使用 `configs.TokenConfig` 进行配置。你需要确保你的配置文件或环境变量中包含以下信息：

- `SigningKey`: 用于签名和验证令牌的密钥
- `ExpiresTime`: 访问令牌的有效期
- `RefreshExpiresTime`: 刷新令牌的有效期
- `Issuer`: 令牌的签发者
- `TokenLookup`: 指定从哪里提取令牌（例如："header:Authorization"）
- `Method`: 签名方法（支持"HS256"、"HS384"、"HS512"）

## 主要功能

### 创建中间件实例

使用 `NewJwt()` 函数创建一个新的JWT中间件实例：

```go
jwtMiddleware, err := auth.NewJwt()
if err != nil {
    // 处理错误
}
```

### 生成令牌对

使用 `GenerateTokenPair()` 方法生成一个包含访问令牌和刷新令牌的令牌对：

```go
tokenPair, err := jwtMiddleware.GenerateTokenPair("username")
if err != nil {
    // 处理错误
}
fmt.Printf("访问令牌: %s\n", tokenPair.AccessToken)
fmt.Printf("刷新令牌: %s\n", tokenPair.RefreshToken)
```

### 验证令牌

使用 `Middleware()` 方法来验证请求中的令牌：

```go
r.Handle("/protected", jwtMiddleware.Middleware(http.HandlerFunc(protectedHandler))).Methods("GET")
```

### 刷新令牌

使用 `RefreshToken()` 方法刷新访问令牌：

```go
newTokenPair, err := jwtMiddleware.RefreshToken(oldRefreshToken)
if err != nil {
    // 处理错误
}
```

### 从上下文获取用户信息

在经过中间件处理的请求中，你可以使用 `GetCurrentUser()` 函数从上下文中获取用户信息：

```go
username, err := auth.GetCurrentUser(r.Context())
if err != nil {
    // 处理错误
}
```

## 高级用法

### 自定义错误处理

你可以通过修改 `JwtConfig` 中的 `ErrHandler` 字段来自定义错误处理：

```go
jwtMiddleware.conf.ErrHandler = func(w http.ResponseWriter, r *http.Request, err error) {
    http.Error(w, "自定义错误: "+err.Error(), http.StatusUnauthorized)
}
```

### 自定义令牌提取方法

默认情况下，中间件支持从请求头、查询参数和cookie中提取令牌。你可以通过修改 `TokenLookup` 配置来自定义提取方法：

```go
config.TokenLookup = "header:X-API-Key" // 从自定义头部提取
```

## 最佳实践

1. 始终使用HTTPS来传输令牌，避免中间人攻击。
2. 设置合理的令牌过期时间，平衡安全性和用户体验。
3. 定期轮换签名密钥。
4. 在生产环境中使用足够长和复杂的密钥。
5. 妥善保管刷新令牌，它们通常有更长的有效期。

## 完整示例

以下是一个更完整的示例，展示了如何在一个简单的Web应用中使用JWT中间件：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"

	"github.com/gorilla/mux"
	"github.com/sagoo-cloud/nexframe/auth"
)

var jwtMiddleware *auth.JwtMiddleware

func main() {
	var err error
	jwtMiddleware, err = auth.NewJwt()
	if err != nil {
		log.Fatalf("创建JWT中间件失败: %v", err)
	}

	r := mux.NewRouter()

	r.HandleFunc("/login", loginHandler).Methods("POST")
	r.HandleFunc("/refresh", refreshHandler).Methods("POST")
	r.Handle("/protected", jwtMiddleware.Middleware(http.HandlerFunc(protectedHandler))).Methods("GET")

	log.Println("服务器启动在 :8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
	// 这里应该有实际的用户验证逻辑
	username := r.FormValue("username")
	password := r.FormValue("password")

	if username == "admin" && password == "password" {
		tokenPair, err := jwtMiddleware.GenerateTokenPair(username)
		if err != nil {
			http.Error(w, "生成令牌失败", http.StatusInternalServerError)
			return
		}

		json.NewEncoder(w).Encode(tokenPair)
	} else {
		http.Error(w, "无效的凭证", http.StatusUnauthorized)
	}
}

func refreshHandler(w http.ResponseWriter, r *http.Request) {
	refreshToken := r.FormValue("refresh_token")
	if refreshToken == "" {
		http.Error(w, "缺少刷新令牌", http.StatusBadRequest)
		return
	}

	newTokenPair, err := jwtMiddleware.RefreshToken(refreshToken)
	if err != nil {
		http.Error(w, "刷新令牌失败", http.StatusUnauthorized)
		return
	}

	json.NewEncoder(w).Encode(newTokenPair)
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
	username, err := auth.GetCurrentUser(r.Context())
	if err != nil {
		http.Error(w, "未授权", http.StatusUnauthorized)
		return
	}
	fmt.Fprintf(w, "欢迎, %s!", username)
}
```

## 常见问题解答

Q: 如何处理令牌过期？
A: 当访问令牌过期时，使用刷新令牌调用 `RefreshToken()` 方法获取新的令牌对。

Q: 可以在令牌中包含自定义信息吗？
A: 目前，令牌中只包含用户名和标准的JWT声明。如果需要包含更多信息，你需要修改 `TokenClaims` 结构体和相关的生成、解析逻辑。

Q: 如何在测试中模拟认证？
A: 你可以使用 `GenerateTokenPair()` 函数创建一个有效的令牌对，然后在测试请求的头部中包含访问令牌。

Q: 中间件是否线程安全？
A: 是的，中间件的设计考虑了并发使用的情况，可以安全地在多个 goroutine 中使用。