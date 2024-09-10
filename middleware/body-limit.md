# Body Limit中间件开发手册

## 1. 简介

Body Limit中间件是一个用于限制HTTP请求体的大小。这个中间件可以帮助您防止因为过大的请求体而导致的服务器资源耗尽问题,增强应用程序的安全性和稳定性。

主要特点:
- 可配置的请求体大小限制
- 支持人性化的大小表示法(如"5M"代表5兆字节)
- 基于Content-Length头和实际读取的内容进行双重检查

## 2. 安装

要使用Body Limit中间件,首先确保您的项目中已经安装了nexframe。然后,您可以直接将中间件代码复制到您的项目中,或者如果它是一个独立的包,可以使用go get命令安装:

```bash
go get github.com/sagoo-cloud/nexframe
```

## 3. 基本用法

以下是一个基本的使用示例:

```go
package main

import (
    "net/http"
    "github.com/gorilla/mux"
    "github.com/sagoo-cloud/nexframe/middleware"
)

func main() {
    r := mux.NewRouter()
    
    // 应用中间件,限制请求体大小为10MB
    r.Use(middleware.BodyLimit("10M"))
    
    // 设置路由
    r.HandleFunc("/upload", uploadHandler).Methods("POST")
    
    http.ListenAndServe(":8080", r)
}

func uploadHandler(w http.ResponseWriter, r *http.Request) {
    // 处理上传逻辑
}
```

在这个例子中,所有通过这个路由器的请求都将被限制为最大10MB的请求体。

## 4. 配置选项

```go

BodyLimitConfig struct {
  // Skipper 定义一个用于跳过中间件的函数。
  Skipper Skipper

  // 请求体的最大允许大小，可以指定为 `4x` 或 `4xB`，
  // 其中 x 是从 K、M、G、T 或 P 中选择的倍数之一。 
  Limit string `json:"limit"`
}


```

Body Limit中间件支持以下配置选项:

- Limit (string): 指定最大允许的请求体大小。支持的单位包括:
    - B: 字节
    - K: 千字节
    - M: 兆字节
    - G: 吉字节
    - T: 太字节
    - P: 拍字节

例如: "5M" 表示5兆字节, "500K" 表示500千字节。

## 5. 工作原理

1. 当请求到达时,中间件首先检查Content-Length头。如果Content-Length超过设定的限制,请求会被立即拒绝,返回413 Request Entity Too Large状态码。

2. 如果Content-Length在限制之内或未设置,中间件会使用自定义的limitedReader包装原始的请求体。

3. limitedReader在读取数据时会累计已读取的字节数。如果在读取过程中超过了限制,会返回一个错误,导致请求被拒绝。

4. 如果整个请求体都在限制之内,请求会正常传递给下一个处理器。

## 6. 高级用法

### 6.1 动态限制

您可以根据不同的路由设置不同的限制:

```go
r := mux.NewRouter()

// 为/upload路由设置100MB的限制
r.Handle("/upload", middleware.BodyLimit("100M")(uploadHandler)).Methods("POST")

// 为/message路由设置1MB的限制
r.Handle("/message", middleware.BodyLimit("1M")(messageHandler)).Methods("POST")
```

### 6.2 条件应用

如果您只想在某些条件下应用限制,可以创建一个自定义的中间件包装器:

```go
func conditionalBodyLimit(limit string) mux.MiddlewareFunc {
    limitMiddleware := bodylimit.BodyLimit(limit)
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if shouldApplyLimit(r) {
                limitMiddleware(next).ServeHTTP(w, r)
            } else {
                next.ServeHTTP(w, r)
            }
        })
    }
}

func shouldApplyLimit(r *http.Request) bool {
    // 实现您的逻辑
    return true
}
```

## 7. 性能考虑

Body Limit中间件使用sync.Pool来重用limitedReader对象,这有助于减少内存分配和垃圾回收的压力,特别是在高并发场景下。

然而,对于非常大的文件上传,您可能需要考虑使用流式处理或分块上传来避免将整个文件加载到内存中。

## 8. 常见问题解答

Q: 如果客户端没有设置Content-Length头,中间件还能正常工作吗?
A: 是的,中间件会在实际读取请求体时检查大小,即使没有Content-Length头也能正常工作。

Q: 中间件会影响文件上传的性能吗?
A: 对于小型文件,影响是微不足道的。对于大型文件,由于需要将整个文件读入内存,可能会对性能产生一定影响。在这种情况下,考虑使用流式上传或增加限制。

## 9. 故障排除

如果您在使用中间件时遇到问题,请检查以下几点:

1. 确保限制值的格式正确(例如"5M", "10K"等)。
2. 检查是否有其他中间件干扰了请求体的读取。
3. 验证客户端是否正确设置了Content-Length头(虽然不是必需的)。

如果问题依然存在,请查看服务器日志以获取更多信息。
