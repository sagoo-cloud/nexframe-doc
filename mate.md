# Meta 处理机制使用说明

## 概述

Meta 处理机制是一个灵活的元数据管理功能，允许开发人员为 API 请求结构添加丰富的元数据。这个功能支持标准的 API 属性（如路径、方法、摘要和标签），同时也允许添加自定义元数据。

## 基本用法

### 1. 在请求结构中嵌入 Meta

在您的 API 请求结构中嵌入 `framework.Meta` 类型，并使用结构体标签来定义元数据：

```go
import "your/framework/package"

type MyRequest struct {
    framework.Meta `path:"/api/example" method:"GET" summary:"Example API" tags:"example,api" custom:"value"`
    // 其他字段...
}
```

### 2. 标准元数据字段

以下是标准的元数据字段：

- `path`: API 的路径
- `method`: HTTP 方法（GET, POST, PUT, DELETE 等）
- `summary`: API 的简短描述
- `tags`: 用于分组和组织 API 的标签（逗号分隔）

### 3. 自定义元数据

您可以添加任何自定义元数据字段，例如：

```go
type MyRequest struct {
    framework.Meta `path:"/api/example" method:"GET" summary:"Example API" tags:"example,api" 
                    version:"1.0" auth:"required" rate-limit:"100/hour"`
    // 其他字段...
}
```

## 高级用法

### 1. 在运行时获取元数据

使用 `GetMetaData` 函数来获取结构体的所有元数据：

```go
metadata := framework.GetMetaData(MyRequest{})
fmt.Println(metadata) // 输出所有元数据
```

### 2. 获取特定的元数据值

使用 `GetMetaValue` 函数来获取特定的元数据值：

```go
path := framework.GetMetaValue(MyRequest{}, "path")
fmt.Println(path) // 输出: /api/example
```

### 3. 在框架中使用元数据

框架会自动处理标准元数据字段（path, method, summary, tags）。所有其他自定义元数据将存储在 `APIDefinition.Meta.ExtraMetadata` 中，可以在需要时访问。

## 最佳实践

1. **保持一致性**：为所有 API 使用一致的元数据格式和命名约定。

2. **文档化自定义元数据**：如果您使用自定义元数据字段，确保在项目文档中明确说明它们的用途和预期值。

3. **避免过度使用**：虽然元数据系统非常灵活，但应该谨慎使用。只添加真正需要的元数据。

4. **验证元数据**：在框架级别实现元数据验证，以确保所有必需的字段都已提供，并且值是有效的。

5. **利用元数据生成文档**：考虑使用元数据来自动生成 API 文档或 Swagger 规范。

## 示例

### 完整的 API 定义示例

```go
type CreateUserRequest struct {
    framework.Meta `path:"/api/users" method:"POST" summary:"Create a new user" 
                    tags:"users,create" auth:"required" rate-limit:"10/minute"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

func (c *UserController) CreateUser(ctx context.Context, req *CreateUserRequest) (*CreateUserResponse, error) {
    // 实现创建用户的逻辑
}
```

在这个示例中，我们定义了一个创建用户的 API，包括标准元数据和自定义元数据（auth 和 rate-limit）。

## 结论

新的 Meta 处理机制提供了一种强大而灵活的方式来管理 API 元数据。通过正确使用这个功能，您可以创建更具描述性和自我文档化的 API，同时为框架提供足够的信息来自动化路由、认证、限流等功能。