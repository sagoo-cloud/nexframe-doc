# CSRF 跨站请求伪造中间件

## 1. 简介

CSRF（跨站请求伪造）是一种常见的 Web 应用程序安全漏洞。本 CSRF 中间件为 NexFrame框架提供了一种简单而有效的方法来防止 CSRF 攻击。它通过生成、验证和管理 CSRF 令牌来保护您的应用程序免受未经授权的请求。

## 2. 安装

首先，确保您的项目中已经安装了 Gorilla Mux。如果没有，可以通过以下命令安装：

```bash
go get -u github.com/sagoo-cloud/nexframe
```

然后，将 CSRF 中间件的代码文件添加到您的项目中。

## 3. 基本用法

### 3.1 使用默认配置

最简单的使用方式是使用默认配置：

```go
package main

import (
    "net/http"
    "github.com/gorilla/mux"
    "github.com/sagoo-cloud/nexframe/middleware"
)

func main() {
	srv := nexframe.Server()
    
    // 使用默认配置的 CSRF 中间件
	srv.WithMiddleware(middleware.CSRF())
	
	// 注册控制器
	err := srv.BindHandlerFunc("/", homeHandler).Methods("GET")
	if err != nil {
		return
	}
	err := srv.BindHandlerFunc("/", submitHandler).Methods("POST")
	if err != nil {
		return
	}
	srv.SetPort(":8080")
	srv.Run()
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    // 从上下文中获取 CSRF 令牌
    token := r.Context().Value("csrf").(string)
    // 在表单中使用此令牌
    html := `
    <form method="POST" action="/submit">
        <input type="hidden" name="csrf_token" value="` + token + `">
        <input type="submit" value="提交">
    </form>`
    w.Write([]byte(html))
}

func submitHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("表单提交成功！"))
}
```

### 3.2 使用自定义配置

如果您需要更细粒度的控制，可以使用自定义配置：

```go
config := middleware.CSRFConfig{
    TokenLength:    32,
    TokenLookup:    "form:csrf_token,header:X-CSRF-Token",
    CookieName:     "my_csrf_token",
    CookieMaxAge:   3600,
    CookieSecure:   true,
    CookieHTTPOnly: true,
}

srv.WithMiddleware(middleware.CSRFWithConfig(config))
```

## 4. 配置选项

`CSRFConfig` 结构体提供了以下配置选项：

- `Skipper`: 函数类型，用于定义跳过中间件处理的条件。
- `TokenLength`: uint8，生成的令牌的长度。
- `TokenLookup`: string，定义如何从请求中提取令牌。
- `ContextKey`: string，用于将生成的 CSRF 令牌存储到上下文中的键。
- `CookieName`: string，CSRF cookie 的名称。
- `CookieDomain`: string，CSRF cookie 的域。
- `CookiePath`: string，CSRF cookie 的路径。
- `CookieMaxAge`: int，CSRF cookie 的最大年龄（以秒为单位）。
- `CookieSecure`: bool，指示 CSRF cookie 是否安全。
- `CookieHTTPOnly`: bool，指示 CSRF cookie 是否为 HTTP only。
- `CookieSameSite`: http.SameSite，指示 CSRF cookie 的 SameSite 模式。
- `ErrorHandler`: 函数类型，用于自定义错误处理。

## 5. 高级用法

### 5.1 自定义令牌提取

您可以自定义从请求中提取令牌的方式：

```go
config := middleware.CSRFConfig{
    TokenLookup: "header:X-CSRF-Token,form:csrf_token,query:csrf_token",
}
```

这将依次从 header、form 和 query 参数中查找令牌。

### 5.2 自定义错误处理

您可以自定义 CSRF 验证失败时的错误处理：

```go
config := middleware.CSRFConfig{
    ErrorHandler: func(err error, w http.ResponseWriter, r *http.Request) {
        http.Error(w, "CSRF 验证失败", http.StatusForbidden)
    },
}
```

### 5.3 跳过特定请求

使用 `Skipper` 函数可以跳过对特定请求的 CSRF 保护：

```go
config := middleware.CSRFConfig{
    Skipper: func(r *http.Request) bool {
        return r.URL.Path == "/public-api"
    },
}
```

## 6. 最佳实践

1. 始终在敏感操作（如表单提交、更改密码等）中使用 CSRF 保护。
2. 将 CSRF 令牌作为隐藏字段包含在所有表单中。
3. 对于 AJAX 请求，将令牌包含在自定义 HTTP 头中。
4. 使用 HTTPS 来加密所有通信，包括 CSRF 令牌。
5. 定期轮换 CSRF 令牌，特别是在用户登录或会话更新时。
6. 确保 `CookieSecure` 和 `CookieHTTPOnly` 在生产环境中设置为 true。

## 7. 故障排除

1. 如果表单提交被拒绝，检查 CSRF 令牌是否正确包含在请求中。
2. 确保 cookie 设置正确，特别是在使用自定义域或路径时。
3. 对于 SPA（单页应用程序），确保在每次页面加载时更新 CSRF 令牌。
4. 如果使用了自定义的 `TokenLookup`，确保它与您的前端代码匹配。

## 8. 示例：在 AJAX 请求中使用 CSRF

```go
// 后端代码

err := srv.BindHandlerFunc("/api/data", submitHandler).Methods("POST")
if err != nil {
return
}
srv

// 前端 JavaScript 代码
fetch('/api/data', {
    method: 'POST',
    headers: {
        'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]').getAttribute('content')
    },
    body: JSON.stringify({ key: 'value' })
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

## 9. 结语

CSRF 中间件为您的应用提供了一个强大而灵活的 CSRF 保护层。通过正确配置和使用这个中间件，您可以显著提高应用的安全性。记住，安全是一个持续的过程，定期审查和更新您的安全策略是很重要的。