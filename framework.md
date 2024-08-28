# API框架手册


## API核心处理框架说明
实现自定义API框架与静态文件处理
实现了一个灵活、可扩展的API框架，集成了静态文件处理、中间件支持和Swagger文档生成功能。主要特性包括：

1. 核心框架结构（APIFramework）:
    - 支持动态注册控制器和API路由
    - 集成Swagger规范生成
    - 灵活的中间件系统

2. 静态文件处理:
    - 实现了自定义StaticHandler，支持多种文件系统（fs.FS接口）
    - 支持设置静态目录（SetStaticDir）和Web根目录（SetWebRoot）

3. 中间件支持:
    - 使用WithMiddleware方法添加全局中间件
    - 支持多个中间件的链式调用

4. API注册和路由:
    - 自动发现和注册API（discoverAPIs方法）
    - 支持路由前缀设置

5. Swagger集成:
    - 自动生成Swagger JSON规范
    - 提供Swagger UI访问路由

6. 灵活的配置:
    - 支持自定义文件系统（SetFileSystem方法）
    - 链式调用API，提高代码可读性

# Framework 自动注入功能

## 概述

我们的 API 框架提供了一个自动注入 Framework 实例的功能。这允许控制器轻松访问框架的功能，而无需手动传递框架实例。

## 工作原理

1. 当注册控制器时，框架会自动检查控制器结构体中是否存在类型为 `*APIFramework` 的字段。
2. 如果找到这样的字段，且它是可导出的（首字母大写），框架会自动将自身的实例注入到这个字段中。
3. 如果没有找到合适的字段，框架会跳过注入过程，不会产生错误。

## 如何使用

### 1. 定义控制器

在您的控制器结构体中，添加一个类型为 `*APIFramework` 的字段。字段名可以自定义，但必须首字母大写：

```go
type UserController struct {
    Framework *APIFramework // 将被自动注入
    // 或者任何其他首字母大写的名称
    // MyFramework *APIFramework // 这同样会被注入
}
```

### 2. 注册控制器

正常注册您的控制器，无需额外步骤：

```go
func main() {
    framework := apiframework.NewAPIFramework()
    userController := &UserController{}
    framework.RegisterController("/api/users", userController)
    // ... 其他设置
}
```

### 3. 在控制器方法中使用

现在您可以在控制器的方法中使用注入的框架字段访问框架功能：

```go
func (c *UserController) GetUser(ctx context.Context, id string) (*User, error) {
    // 使用框架功能，例如获取其他控制器
    otherController, ok := c.Framework.GetController("OtherController")
    // 或者 c.APIFramework.GetController，取决于您如何命名字段
    if ok {
        // 使用其他控制器
    }
    // ... 实现方法逻辑
}
```

## 注意事项

1. **字段类型**：框架字段必须是 `*APIFramework` 类型。
2. **字段可见性**：字段必须是可导出的（首字母大写）。
3. **可选性**：这个功能是可选的。如果您不需要在控制器中访问框架，可以不包含 `*APIFramework` 类型的字段。
4. **多个字段**：如果存在多个符合条件的字段，框架会注入到它找到的第一个字段中。
5. **初始化顺序**：框架实例会在控制器注册时被注入，因此在构造函数或 `init` 方法中，该字段可能还是 nil。

## 最佳实践

1. 为了一致性，建议在团队内统一字段的命名约定，例如总是使用 `Framework` 或 `APIFramework`。
2. 仅在需要访问框架功能的控制器中添加 `*APIFramework` 类型的字段。
3. 使用依赖注入模式可以使您的代码更容易测试。在单元测试中，您可以注入一个模拟的框架实例。
4. 避免在控制器之间创建复杂的依赖关系。如果发现多个控制器频繁相互调用，考虑重构以提取共享逻辑到服务层。

## 示例

```go
type UserController struct {
    Framework *APIFramework // 将被自动注入
}

func (c *UserController) CreateUser(ctx context.Context, req *CreateUserRequest) (*User, error) {
    // 使用框架功能
    authController, ok := c.Framework.GetController("AuthController")
    if !ok {
        return nil, errors.New("auth controller not found")
    }

    // 调用其他控制器的方法
    isAuthorized := authController.(*AuthController).CheckAuthorization(ctx, req.Token)
    if !isAuthorized {
        return nil, errors.New("unauthorized")
    }

    // ... 创建用户的逻辑
    return &User{ID: "new-user-id", Name: req.Name}, nil
}
```

通过遵循这些指南，您可以有效地利用框架的自动注入功能，使您的控制器代码更加清晰和模块化。