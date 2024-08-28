# HTTP 客户端工具包使用手册

## 1. 简介

本 HTTP 客户端工具包提供了一套简单易用的 HTTP 请求接口，支持 GET、POST（表单和 JSON）等常见的 HTTP 方法。该工具包旨在简化 HTTP 请求的创建和发送过程，并提供了一些实用的功能，如超时设置和请求头管理。

## 2. 安装

确保您的项目使用的是 Go 模块。在您的项目中导入此包：

```go
import "github.com/sagoo-cloud/nexframe/httputil"
```

请将 "path/to" 替换为实际的包路径。

## 3. 主要功能

### 3.1 创建客户端

使用 `NewClient` 函数创建一个新的 HTTP 客户端：

```go
client := httputil.NewClient(30 * time.Second)
```

这将创建一个超时时间为 30 秒的客户端。

### 3.2 发送 GET 请求

使用 `Get` 函数发送 GET 请求：

```go
resp, err := httputil.Get(ctx, "http://example.com", map[string]interface{}{
    "param1": "value1",
    "param2": "value2",
})
```

### 3.3 发送 POST 请求（表单数据）

使用 `Post` 函数发送带有表单数据的 POST 请求：

```go
resp, err := httputil.Post(ctx, "http://example.com", map[string]interface{}{
    "key1": "value1",
    "key2": []string{"value2", "value3"},
})
```

### 3.4 发送 POST 请求（JSON 数据）

使用 `PostJSON` 函数发送带有 JSON 数据的 POST 请求：

```go
resp, err := httputil.PostJSON(ctx, "http://example.com", map[string]interface{}{
    "key1": "value1",
    "key2": []int{1, 2, 3},
})
```

### 3.5 自定义请求

使用 `Request` 函数发送自定义的 HTTP 请求：

```go
resp, err := httputil.Request(ctx, "PUT", "http://example.com", map[string]interface{}{
    "key": "value",
})
```

### 3.6 处理响应

使用 `DealResponse` 函数读取和关闭响应体：

```go
body, err := httputil.DealResponse(resp)
if err != nil {
    // 处理错误
}
fmt.Println(string(body))
```

## 4. 高级用法

### 4.1 设置请求头

在发送请求时，可以通过添加额外的参数来设置请求头：

```go
headers := map[string]string{
    "Authorization": "Bearer token123",
}
req, err := httputil.NewGetRequest("http://example.com", nil, headers)
```

### 4.2 设置超时

在使用 `Get`、`Post` 或 `Request` 函数时，可以通过添加选项来设置超时：

```go
options := map[string]interface{}{
    "timeout": 60, // 60 秒
}
resp, err := httputil.Get(ctx, "http://example.com", nil, options)
```

### 4.3 处理大型响应

对于大型响应，可以使用 `io.Copy` 来高效地处理数据：

```go
resp, err := httputil.Get(ctx, "http://example.com/largefile", nil)
if err != nil {
    // 处理错误
}
defer resp.Body.Close()

file, err := os.Create("largefile.dat")
if err != nil {
    // 处理错误
}
defer file.Close()

_, err = io.Copy(file, resp.Body)
if err != nil {
    // 处理错误
}
```

## 5. 错误处理

所有的函数都返回一个 error 类型的第二个返回值。请始终检查这个错误：

```go
resp, err := httputil.Get(ctx, "http://example.com", nil)
if err != nil {
    log.Printf("请求失败: %v", err)
    return
}
```

## 6. 测试示例

以下是一个完整的测试示例，展示了如何使用此工具包：

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"net/http/httptest"
	"path/to/httputil"
	"time"
)

func main() {
	// 创建一个测试服务器
	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.Method == "GET" {
			w.Write([]byte("Hello, GET!"))
		} else if r.Method == "POST" {
			w.Write([]byte("Hello, POST!"))
		}
	}))
	defer ts.Close()

	ctx := context.Background()

	// 测试 GET 请求
	resp, err := httputil.Get(ctx, ts.URL, nil)
	if err != nil {
		log.Fatalf("GET 请求失败: %v", err)
	}
	body, err := httputil.DealResponse(resp)
	if err != nil {
		log.Fatalf("读取 GET 响应失败: %v", err)
	}
	fmt.Printf("GET 响应: %s\n", string(body))

	// 测试 POST 请求
	resp, err = httputil.Post(ctx, ts.URL, map[string]interface{}{"key": "value"})
	if err != nil {
		log.Fatalf("POST 请求失败: %v", err)
	}
	body, err = httputil.DealResponse(resp)
	if err != nil {
		log.Fatalf("读取 POST 响应失败: %v", err)
	}
	fmt.Printf("POST 响应: %s\n", string(body))

	// 测试带超时的请求
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	resp, err = httputil.Get(ctx, ts.URL, nil)
	if err != nil {
		log.Fatalf("带超时的 GET 请求失败: %v", err)
	}
	body, err = httputil.DealResponse(resp)
	if err != nil {
		log.Fatalf("读取带超时的 GET 响应失败: %v", err)
	}
	fmt.Printf("带超时的 GET 响应: %s\n", string(body))
}
```

这个示例创建了一个测试服务器，然后使用我们的 HTTP 客户端工具包发送 GET 和 POST 请求，以及一个带有超时的请求。它展示了如何使用不同的函数，以及如何处理响应和错误。

## 7. 注意事项

- 始终记得关闭响应体，最好使用 `defer resp.Body.Close()`。
- 对于大型请求或响应，考虑使用流式处理而不是将整个内容加载到内存中。
- 在生产环境中，请确保正确处理所有可能的错误情况。
- 如果您的应用程序需要频繁进行 HTTP 请求，考虑重用 `Client` 实例而不是每次都创建新的。

