# Homedir 库使用手册

## 1. 简介

Homedir 是一个 Go 语言库，提供了跨平台获取和操作用户主目录的功能。这个库主要解决了在不同操作系统上统一获取用户主目录的问题，并提供了一些便利的功能，如展开以 `~` 开头的路径。

## 2. 功能特点

- 跨平台支持（Windows, Unix-like 系统，包括 macOS）
- 获取当前用户的主目录
- 展开以 `~` 开头的路径
- 结果缓存，提高性能
- 支持缓存重置

## 3. 安装

要使用 Homedir 库，首先需要将其添加到你的 Go 项目中：

```
go get github.com/sagoo-cloud/nexframe
```

## 4. 基本用法

### 4.1 获取用户主目录

使用 `Dir()` 函数来获取当前用户的主目录：

```go
import "github.com/sagoo-cloud/nexframe/os/homedir"

func main() {
    dir, err := homedir.Dir()
    if err != nil {
        panic(err)
    }
    fmt.Println("Home directory:", dir)
}
```

### 4.2 展开以 `~` 开头的路径

使用 `Expand()` 函数来展开以 `~` 开头的路径：

```go
path := "~/documents/file.txt"
expandedPath, err := homedir.Expand(path)
if err != nil {
    panic(err)
}
fmt.Println("Expanded path:", expandedPath)
```

### 4.3 重置缓存

如果需要强制重新检测主目录（例如在测试中），可以使用 `Reset()` 函数：

```go
homedir.Reset()
```

## 5. 高级用法

### 5.1 禁用缓存

如果你需要在每次调用时都重新检测主目录，可以禁用缓存：

```go
homedir.disableCache = true
```

注意：禁用缓存可能会影响性能，特别是在频繁调用 `Dir()` 的情况下。

### 5.2 错误处理

Homedir 库定义了几种特定的错误类型，你可以根据这些错误类型来进行更精细的错误处理：

```go
dir, err := homedir.Dir()
if err != nil {
    switch err {
    case homedir.ErrCannotExpandUser:
        fmt.Println("无法展开用户特定的主目录")
    case homedir.ErrBlankOutput:
        fmt.Println("读取主目录时得到空白输出")
    case homedir.ErrMissingWindowsEnv:
        fmt.Println("Windows 环境变量缺失")
    default:
        fmt.Println("未知错误:", err)
    }
}
```

## 6. 最佳实践

1. **错误处理**：始终检查并适当处理 `Dir()` 和 `Expand()` 函数返回的错误。

2. **路径操作**：使用 `filepath` 包来处理路径，以确保跨平台兼容性。

3. **缓存使用**：除非有特殊需求，否则保持缓存启用以提高性能。

4. **测试**：在编写测试时，使用 `Reset()` 函数来确保每次测试都从一个干净的状态开始。

## 7. 示例：创建配置文件

以下是一个使用 Homedir 库来创建用户配置文件的示例：

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"

    "github.com/sagoo-cloud/nexframe/os/homedir"
)

func main() {
    configPath, err := homedir.Expand("~/.myapp/config.json")
    if err != nil {
        fmt.Println("Error expanding path:", err)
        return
    }

    // 确保配置目录存在
    configDir := filepath.Dir(configPath)
    err = os.MkdirAll(configDir, 0755)
    if err != nil {
        fmt.Println("Error creating config directory:", err)
        return
    }

    // 创建配置文件
    file, err := os.Create(configPath)
    if err != nil {
        fmt.Println("Error creating config file:", err)
        return
    }
    defer file.Close()

    // 写入一些默认配置
    _, err = file.WriteString("{\"setting\": \"default\"}")
    if err != nil {
        fmt.Println("Error writing to config file:", err)
        return
    }

    fmt.Println("Config file created successfully at:", configPath)
}
```

这个示例展示了如何使用 Homedir 库来创建一个位于用户主目录下的配置文件。

## 8. 注意事项

1. **权限**：在某些系统上，主目录可能受到权限限制。确保你的应用程序有适当的权限来访问和修改主目录中的文件。

2. **环境变量**：Homedir 库依赖于系统环境变量。在某些受限环境中，这些变量可能不可用或被修改。

3. **跨用户操作**：这个库设计用于获取当前用户的主目录。如果你需要获取其他用户的主目录，可能需要其他方法。

4. **性能考虑**：虽然库使用了缓存来提高性能，但在高并发场景下，可能需要考虑更高效的方法来处理主目录路径。

## 9. 结论

Homedir 库提供了一种简单而强大的方式来处理用户主目录相关的操作，特别是在需要跨平台兼容性的情况下。通过遵循本手册中的指导和最佳实践，你可以轻松地在你的 Go 应用程序中集成和使用这个库。

如果你在使用过程中遇到任何问题或需要进一步的帮助，请查阅相关文档或寻求社区支持。