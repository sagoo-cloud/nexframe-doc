# Secure中间件

## 1. 简介

Secure 中间件是一个为NexFrame框架设计的安全增强组件。它通过设置各种 HTTP 安全头来提高 Web 应用的安全性，包括防止 XSS 攻击、点击劫持、内容类型嗅探等。这个中间件易于集成，高度可配置，适用于各种 Web 应用场景。

## 2. 安装

首先，确保你的项目中已经安装了 NexFrame。如果没有，可以通过以下命令安装：

```bash
go get -u github.com/sagoo-cloud/nexframe
```

然后，将 Secure 中间件的代码文件添加到你的项目中。

## 3. 基本用法

### 3.1 使用默认配置

最简单的使用方式是使用默认配置：

```go
package main

import (
    "net/http"
    "github.com/gorilla/mux"
    "github.com/sagoo-cloud/nexframe/nf/middleware"
)

func main() {
	srv := nexframe.Server()
    
    // 使用默认配置的 Secure 中间件
	srv.WithMiddleware(middleware.Secure())

}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("欢迎来到安全的首页！"))
}
```

### 3.2 使用自定义配置

如果你需要更细粒度的控制，可以使用自定义配置：

```go
package main

import (
    "net/http"
    "github.com/gorilla/mux"
    "github.com/sagoo-cloud/nexframe/nf/middleware"
)

func main() {
	srv := nexframe.Server()
    
    config := middleware.SecureConfig{
        XSSProtection:         "1; mode=block",
        ContentTypeNosniff:    "nosniff",
        XFrameOptions:         "DENY",
        HSTSMaxAge:            31536000,
        ContentSecurityPolicy: "default-src 'self'",
        ReferrerPolicy:        "strict-origin-when-cross-origin",
    }

	srv.WithMiddleware(middleware.SecureWithConfig(config))

	// 注册控制器
	err := srv.BindHandlerFunc("/", homeHandler)
	if err != nil {
		return
	}
	srv.SetPort(":8080")
	srv.Run()
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("欢迎来到安全的首页！"))
}
```

## 4. 配置选项

`SecureConfig` 结构体提供了以下配置选项：

- `Skipper`: 函数类型，用于定义跳过中间件处理的条件。
- `XSSProtection`: 设置 `X-XSS-Protection` 头，防止跨站脚本攻击。
- `ContentTypeNosniff`: 设置 `X-Content-Type-Options` 头，防止内容类型嗅探。
- `XFrameOptions`: 设置 `X-Frame-Options` 头，防止点击劫持。
- `HSTSMaxAge`: 设置 `Strict-Transport-Security` 头的 max-age 值。
- `HSTSExcludeSubdomains`: 是否在 HSTS 头中排除子域。
- `ContentSecurityPolicy`: 设置 `Content-Security-Policy` 头。
- `CSPReportOnly`: 是否使用 `Content-Security-Policy-Report-Only` 头。
- `HSTSPreloadEnabled`: 是否在 HSTS 头中添加 preload 指令。
- `ReferrerPolicy`: 设置 `Referrer-Policy` 头。

## 5. 高级用法

### 5.1 使用 Skipper 函数

你可以使用 Skipper 函数来有选择地跳过某些请求的安全处理：

```go
config := middleware.SecureConfig{
    Skipper: func(r *http.Request) bool {
        return r.URL.Path == "/public"
    },
    // 其他配置...
}

srv.WithMiddleware(middleware.SecureWithConfig(config))
```

### 5.2 配置内容安全策略（CSP）

内容安全策略可以提供额外的安全层：

```go
config := middleware.SecureConfig{
    ContentSecurityPolicy: "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:",
    // 其他配置...
}

srv.WithMiddleware(middleware.SecureWithConfig(config))
```

### 5.3 启用 HSTS Preload

如果你的网站准备加入 HSTS preload 列表，可以这样配置：

```go
config := middleware.SecureConfig{
    HSTSMaxAge:         63072000, // 2年
    HSTSPreloadEnabled: true,
    // 其他配置...
}

srv.WithMiddleware(middleware.SecureWithConfig(config))
```

## 6. 最佳实践

1. 在生产环境中始终使用 HTTPS。
2. 定期审查和更新你的安全配置。
3. 使用强密码和适当的会话管理。
4. 结合其他安全措施，如 CSRF 保护、安全的认证机制等。
5. 监控你的应用，及时发现和处理安全问题。

## 7. 故障排除

1. 如果某些安全头没有设置，检查你的配置是否正确。
2. 使用浏览器的开发者工具检查响应头，确保它们被正确设置。
3. 如果遇到 CORS 问题，可能需要调整你的 `Content-Security-Policy`。
4. 对于旧版浏览器，某些安全头可能不被支持，请进行兼容性测试。

