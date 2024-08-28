# file 包使用说明文档

## 简介

file 包是一个用于处理文件上传和图片水印的 Go 语言工具包。它提供了简单而强大的 API，用于管理文件上传、限制文件类型和大小，以及为图片添加水印。file 包提供了一种简单而灵活的方式来处理文件上传和图片水印。通过合理配置和使用，它可以帮助您快速实现安全可靠的文件处理功能。

## 主要功能

1. 文件上传
2. 文件类型和大小限制
3. 自动生成唯一文件名
4. 图片水印

## 安装

使用以下命令安装 file 包：

```bash
go get github.com/sagoo-cloud/nexframe
```

## 使用方法

### 1. 创建 FileHandler

首先，需要创建一个 `FileHandler` 实例来处理文件上传和水印功能。

```go
import "github.com/sagoo-cloud/nexframe/file"

config := file.UploadConfig{
    Dir:        "/path/to/upload/directory",
    Format:     "2006/01/02/",
    MaxSize:    10 * 1024 * 1024, // 10MB
    AllowedExt: []string{".jpg", ".png", ".gif"},
}

handler, err := file.NewFileHandler(config)
if err != nil {
    // 处理错误
}
```

### 2. 处理文件上传

在 HTTP 处理函数中使用 `Upload` 方法来处理文件上传：

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    results, err := handler.Upload("file", r)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    for _, result := range results {
        if result.Error != nil {
            fmt.Fprintf(w, "文件 %s 上传失败：%v\n", result.Filename, result.Error)
        } else {
            fmt.Fprintf(w, "文件 %s 上传成功\n", result.Filename)
        }
    }
}
```

### 3. 添加水印功能

如果需要为上传的图片添加水印，可以在创建 `FileHandler` 时配置水印：

```go
watermarkConfig := &file.WatermarkConfig{
    Path:    "/path/to/watermark.png",
    Padding: 10,
    Pos:     file.BottomRight,
}

config.Watermark = watermarkConfig

handler, err := file.NewFileHandler(config)
if err != nil {
    // 处理错误
}
```

### 4. 自定义文件命名

file 包默认使用基于时间戳和随机字符串的文件命名策略。如果需要自定义命名策略，可以在创建 `FileHandler` 时提供自定义的 `generateFilename` 函数：

```go
config.GenerateFilename = func(originalName string) string {
    // 自定义文件命名逻辑
    return customFileName
}

handler, err := file.NewFileHandler(config)
if err != nil {
    // 处理错误
}
```

## 配置选项

### UploadConfig

- `Dir`: 文件上传的目标目录
- `Format`: 子目录的日期格式，例如 "2006/01/02/"
- `MaxSize`: 允许上传的最大文件大小（字节）
- `AllowedExt`: 允许上传的文件扩展名列表
- `Watermark`: 水印配置（可选）
- `GenerateFilename`: 自定义文件命名函数（可选）

### WatermarkConfig

- `Path`: 水印图片的路径
- `Padding`: 水印与图片边缘的间距（像素）
- `Pos`: 水印位置，可选值：
  - `file.TopLeft`
  - `file.TopRight`
  - `file.BottomLeft`
  - `file.BottomRight`
  - `file.Center`

## 注意事项

1. 确保上传目录具有适当的写入权限。
2. 合理设置 `MaxSize` 以防止过大的文件上传。
3. 仅允许安全的文件类型上传，避免潜在的安全风险。
4. 水印图片应当是 PNG 格式，以支持透明度。

## 错误处理

file 包使用自定义的 `FileError` 类型来提供详细的错误信息。在使用过程中，建议进行适当的错误处理和日志记录。

## 示例

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/sagoo-cloud/nexframe/file"
)

func main() {
    config := file.UploadConfig{
        Dir:        "./uploads",
        Format:     "2006/01/02/",
        MaxSize:    5 * 1024 * 1024, // 5MB
        AllowedExt: []string{".jpg", ".png", ".gif"},
        Watermark: &file.WatermarkConfig{
            Path:    "./watermark.png",
            Padding: 10,
            Pos:     file.BottomRight,
        },
    }

    handler, err := file.NewFileHandler(config)
    if err != nil {
        panic(err)
    }

    http.HandleFunc("/upload", func(w http.ResponseWriter, r *http.Request) {
        results, err := handler.Upload("file", r)
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }

        for _, result := range results {
            if result.Error != nil {
                fmt.Fprintf(w, "文件上传失败：%v\n", result.Error)
            } else {
                fmt.Fprintf(w, "文件上传成功：%s\n", result.Filename)
            }
        }
    })

    fmt.Println("服务器启动在 :8080 端口")
    http.ListenAndServe(":8080", nil)
}
```

这个示例创建了一个简单的 HTTP 服务器，它可以处理文件上传，并为图片添加水印。

## 
