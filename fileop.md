# 文件操作工具包使用手册

## 1. 概述

本文件操作工具包提供了一系列高效、安全的文件操作函数和方法。它支持基本的文件读写操作，同时还提供了缓存机制、并发控制和大文件处理等高级功能。本包适用于需要频繁进行文件操作或处理大文件的 Go 语言项目。

## 2. 主要特性

- 基本文件操作（读、写、追加、截断等）
- 文件信息缓存机制，提高读取效率
- 并发安全的文件操作
- 支持大文件的分块处理
- 并行文件处理，带并发限制
- 灵活的文件路径处理函数

## 3. 安装

在您的 Go 项目中使用以下命令安装本包：

```bash
go get github.com/sagoo-cloud/nexframe
```

## 4. 初始化

在使用本包之前，建议先进行初始化：

```go
import "github.com/sagoo-cloud/nexframe/file"

func main() {
    file.InitFileSystem()
    // 其他代码...
}
```

`InitFileSystem` 函数会设置默认的缓存过期时间和其他初始化参数。

## 5. 基本使用

### 5.1 创建文件对象

```go
f := file.NewFile("/path/to/your/file.txt")
```

### 5.2 读取文件

```go
// 读取整个文件
content, err := f.ReadAll()
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(content))

// 读取一行
line, err := f.ReadLine()
if err != nil {
    log.Fatal(err)
}
fmt.Println(line)

// 读取指定大小的块
block, size, err := f.ReadBlock(1024) // 读取1KB
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Read %d bytes: %s\n", size, string(block))
```

### 5.3 写入文件

```go
// 写入文件（覆盖原内容）
err := f.Write([]byte("Hello, World!"))
if err != nil {
    log.Fatal(err)
}

// 在指定位置写入
n, err := f.WriteAt([]byte("Go"), 7)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Wrote %d bytes\n", n)

// 追加内容
err = f.WriteAppend([]byte("\nAppended content"))
if err != nil {
    log.Fatal(err)
}
```

### 5.4 获取文件信息

```go
// 获取文件大小
size, err := f.Size()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("File size: %d bytes\n", size)

// 获取文件修改时间
modTime, err := f.ModifyTime()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Last modified: %v\n", modTime)

// 获取文件扩展名
ext := f.Ext()
fmt.Printf("File extension: %s\n", ext)
```

## 6. 高级功能

### 6.1 流式读取大文件

```go
err := f.StreamRead(func(chunk []byte) error {
    // 处理每个数据块
    fmt.Printf("Read chunk: %d bytes\n", len(chunk))
    return nil
})
if err != nil {
    log.Fatal(err)
}
```

### 6.2 分块处理大文件

```go
processor := file.ChunkProcessor{
    ChunkSize: 1024 * 1024, // 1MB 块大小
    Handler: func(chunk []byte) error {
        // 处理每个数据块
        fmt.Printf("Processing chunk: %d bytes\n", len(chunk))
        return nil
    },
}

err := file.ProcessLargeFile("/path/to/large/file.dat", processor)
if err != nil {
    log.Fatal(err)
}
```

### 6.3 并行处理多个文件

```go
files := []string{
    "/path/to/file1.txt",
    "/path/to/file2.txt",
    "/path/to/file3.txt",
}

err := file.ProcessFilesInParallel(files, func(filePath string) error {
    // 处理每个文件
    fmt.Printf("Processing file: %s\n", filePath)
    return nil
}, 5) // 最大并发数为 5

if err != nil {
    log.Fatal(err)
}
```

### 6.4 刷新文件缓存

```go
// 刷新单个文件的缓存
err := file.RefreshFileCache("/path/to/file.txt")
if err != nil {
    log.Fatal(err)
}

// 刷新所有文件缓存
file.RefreshAllFileCache()
```

### 6.5 设置缓存过期时间

```go
file.SetCacheExpiration(10 * time.Minute)
```

## 7. 注意事项

1. 并发安全：本包的所有文件操作都是并发安全的，但在处理同一文件时仍需注意避免竞态条件。

2. 缓存一致性：虽然包内部会在写操作后自动刷新缓存，但如果文件被外部程序修改，可能需要手动调用 `RefreshFileCache` 或 `RefreshAllFileCache`。

3. 大文件处理：对于非常大的文件，建议使用 `StreamRead` 或 `ProcessLargeFile` 方法，避免一次性将整个文件加载到内存。

4. 错误处理：所有可能出错的操作都会返回错误，请确保适当处理这些错误。

5. 资源管理：本包会自动管理文件句柄的打开和关闭，但在长时间运行的程序中，建议定期调用 `RefreshAllFileCache` 以释放不再需要的资源。

## 8. 性能优化建议

1. 对于频繁访问的小文件，可以利用缓存机制提高性能。

2. 处理大量小文件时，使用 `ProcessFilesInParallel` 可以显著提高效率，但要注意设置合适的并发限制。

3. 对于大文件，根据实际情况选择合适的块大小，过大的块可能导致内存压力，过小的块可能影响处理效率。

4. 在写入操作频繁的场景下，可以考虑降低缓存刷新频率，以减少磁盘 I/O。

## 9. 示例应用

以下是一个综合使用本包功能的示例应用，用于处理日志文件：

```go
package main

import (
    "fmt"
    "log"
    "github.com/sagoo-cloud/nexframe/file"
)

func main() {
    // 初始化文件系统
    file.InitFileSystem()

    // 设置缓存过期时间
    file.SetCacheExpiration(30 * time.Minute)

    // 处理单个大日志文件
    logFile := "/var/log/app.log"
    processor := file.ChunkProcessor{
        ChunkSize: 1024 * 1024, // 1MB 块大小
        Handler: func(chunk []byte) error {
            // 这里可以添加日志处理逻辑，如解析、统计等
            fmt.Printf("Processed %d bytes of log data\n", len(chunk))
            return nil
        },
    }

    err := file.ProcessLargeFile(logFile, processor)
    if err != nil {
        log.Fatalf("Failed to process log file: %v", err)
    }

    // 并行处理多个配置文件
    configFiles := []string{
        "/etc/app/config1.json",
        "/etc/app/config2.json",
        "/etc/app/config3.json",
    }

    err = file.ProcessFilesInParallel(configFiles, func(filePath string) error {
        f := file.NewFile(filePath)
        content, err := f.ReadAll()
        if err != nil {
            return fmt.Errorf("failed to read config file %s: %v", filePath, err)
        }
        fmt.Printf("Config file %s content: %s\n", filePath, string(content))
        return nil
    }, 3) // 最大并发数为 3

    if err != nil {
        log.Fatalf("Failed to process config files: %v", err)
    }

    // 写入汇总报告
    report := file.NewFile("/var/log/summary.txt")
    err = report.WriteAppend([]byte("Log processing completed successfully.\n"))
    if err != nil {
        log.Fatalf("Failed to write summary: %v", err)
    }

    fmt.Println("Application completed successfully.")
}
```

这个示例展示了如何使用本包处理大型日志文件、并行读取多个配置文件，以及写入汇总报告。它涵盖了文件的读取、写入、大文件处理和并行处理等多个方面。

## 

本文件操作工具包提供了丰富的功能和灵活的使用方式，可以满足多种文件操作场景的需求。在使用过程中，请注意参考本手册中的注意事项和建议，以确保最佳的性能和可靠性。