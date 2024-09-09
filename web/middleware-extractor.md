# Extractor 开发手册

## 简介

Extractor 是一种用于从 HTTP 请求中提取特定值的功能组件。它们通常用于身份验证、授权或其他需要从请求中获取特定信息的场景。本手册将指导您如何创建、使用和测试 Extractor。

## Extractor 的基本结构

Extractor 的核心是一个函数类型，定义如下：

```go
type ValuesExtractor func(r *http.Request) ([]string, error)
```

这个函数接收一个 `*http.Request` 参数，返回一个字符串切片和一个错误。字符串切片包含提取的值，错误用于指示提取过程中是否发生了问题。

## 创建自定义 Extractor

创建自定义 Extractor 的基本步骤如下：

1. 定义一个符合 `ValuesExtractor` 类型的函数。
2. 在函数内部，从请求中提取所需的值。
3. 返回提取的值和可能的错误。

示例：从自定义 header 中提取值

```go
func valueFromCustomHeader(headerName string) ValuesExtractor {
    return func(r *http.Request) ([]string, error) {
        value := r.Header.Get(headerName)
        if value == "" {
            return nil, fmt.Errorf("header %s not found", headerName)
        }
        return []string{value}, nil
    }
}
```

## 内置 Extractor 类型

我们的中间件包提供了几种内置的 Extractor 类型：

1. `valuesFromHeader`: 从请求头中提取值
2. `valuesFromQuery`: 从查询参数中提取值
3. `valuesFromParam`: 从 URL 路径参数中提取值
4. `valuesFromCookie`: 从 Cookie 中提取值
5. `valuesFromForm`: 从表单数据中提取值

示例：使用内置 Extractor

```go
headerExtractor := valuesFromHeader("Authorization", "Bearer ")
queryExtractor := valuesFromQuery("token")
paramExtractor := valuesFromParam("id")
cookieExtractor := valuesFromCookie("session")
formExtractor := valuesFromForm("csrf_token")
```

## 使用 CreateExtractors 函数

`CreateExtractors` 函数允许您通过字符串配置创建多个 Extractor：

```go
extractors, err := CreateExtractors("header:Authorization,query:token,param:id")
if err != nil {
    // 处理错误
}
```

这个函数返回一个 Extractor 切片，可以直接传递给中间件。

## 在中间件中使用 Extractor

Extractor 通常在中间件中使用。以下是一个使用 Extractor 的中间件示例：

```go
func ExtractorMiddleware(extractors ...ValuesExtractor) mux.MiddlewareFunc {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            for _, extractor := range extractors {
                values, err := extractor(r)
                if err == nil && len(values) > 0 {
                    ctx := context.WithValue(r.Context(), ExtractedValuesKey, values)
                    r = r.WithContext(ctx)
                    break
                }
            }
            next.ServeHTTP(w, r)
        })
    }
}
```

使用示例：

```go
	srv := nexframe.Server()
extractors, _ := CreateExtractors("header:Authorization,query:token")
srv.WithMiddleware(ExtractorMiddleware(extractors...))
```

## 测试 Extractor

测试 Extractor 时，您需要模拟 HTTP 请求并验证提取的值。以下是一个测试示例：

```go
func TestHeaderExtractor(t *testing.T) {
    req := httptest.NewRequest("GET", "/", nil)
    req.Header.Set("Authorization", "Bearer token123")

    extractor := valuesFromHeader("Authorization", "Bearer ")
    values, err := extractor(req)

    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
    if len(values) != 1 || values[0] != "token123" {
        t.Errorf("Expected [token123], got %v", values)
    }
}
```

## 最佳实践和注意事项

1. 错误处理：始终返回明确的错误，以便调试和日志记录。
2. 安全性：在处理敏感数据（如认证令牌）时，要格外小心。
3. 性能：Extractor 应该高效，避免不必要的计算或内存分配。
4. 可测试性：设计 Extractor 时考虑到易于测试。
5. 灵活性：创建可重用和可配置的 Extractor。
6. 文档：为自定义 Extractor 提供清晰的文档和使用示例。

遵循这些指南，您可以创建强大、灵活且易于维护的 Extractor。