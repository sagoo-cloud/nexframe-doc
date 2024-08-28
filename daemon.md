# Daemon 服务管理器使用手册

## 1. 简介

Daemon 服务管理器是提供了将应用程序作为系统服务（守护进程）运行的功能。这个库封装了服务管理的核心功能，使得开发者可以轻松地将他们的应用程序转换为系统服务，并进行管理。

## 2. 功能特点

- 创建和管理系统服务
- 将普通应用程序转换为守护进程
- 支持服务的启动、停止和重启操作
- 跨平台支持（Windows, Linux, macOS）

## 3. 安装

首先，确保你的项目中已经安装了必要的依赖：

```
go get github.com/kardianos/service
```

然后，将 Daemon 服务管理器库添加到你的项目中（假设该库已发布）：

```
go get github.com/sagoo-cloud/nexframe
```

## 4. 基本用法

### 4.1 创建服务管理器

要使用 Daemon 服务管理器，首先需要创建一个 `ServiceManager` 实例：

```go
import (
    "github.com/sagoo-cloud/nexframe/os/daemon"
)

func main() {
    sm, err := daemon.NewServiceManager("MyService", "My Service Display Name", "This is my service description")
    if err != nil {
        panic(err)
    }

    // 使用 sm 进行后续操作
}
```

### 4.2 运行服务

使用 `Run` 方法来运行服务：

```go
err := sm.Run()
if err != nil {
    panic(err)
}
```

### 4.3 控制服务

使用 `Control` 方法来控制服务的状态：

```go
// 启动服务
err := sm.Control("start")

// 停止服务
err := sm.Control("stop")

// 重启服务
err := sm.Control("restart")
```

### 4.4 将应用程序转换为守护进程

使用 `Daemonize` 函数将当前程序作为守护进程运行：

```go
err := daemon.Daemonize()
if err != nil {
    panic(err)
}
```

### 4.5 检查是否作为守护进程运行

使用 `IsDaemon` 函数检查当前进程是否作为守护进程运行：

```go
if daemon.IsDaemon() {
    fmt.Println("Running as a daemon")
} else {
    fmt.Println("Not running as a daemon")
}
```

## 5. 高级用法

### 5.1 自定义服务行为

你可以通过实现自己的 `program` 结构体来自定义服务的行为：

```go
type MyProgram struct {
    exit chan struct{}
}

func (p *MyProgram) Start(s service.Service) error {
    go p.run()
    return nil
}

func (p *MyProgram) Stop(s service.Service) error {
    close(p.exit)
    return nil
}

func (p *MyProgram) run() {
    // 实现你的服务逻辑
    for {
        select {
        case <-p.exit:
            return
        default:
            // 执行服务任务
        }
    }
}
```

然后，你可以在创建 `ServiceManager` 时使用这个自定义的 program：

```go
sm, err := daemon.NewServiceManager("MyService", "My Service", "Description")
if err != nil {
    panic(err)
}

myProgram := &MyProgram{exit: make(chan struct{})}
sm.program = myProgram
```

## 6. 最佳实践

1. **错误处理**：总是检查并适当处理返回的错误。

2. **日志记录**：在服务运行时使用适当的日志记录机制，以便于问题诊断。

3. **优雅关闭**：实现优雅的关闭逻辑，确保在服务停止时正确清理资源。

4. **配置管理**：考虑使用配置文件或环境变量来管理服务的配置，而不是硬编码。

5. **权限管理**：确保服务运行时具有适当的系统权限。

## 7. 示例：创建一个简单的后台服务

以下是一个创建简单后台服务的完整示例：

```go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/sagoo-cloud/nexframe/os/daemon"
)

func main() {
    sm, err := daemon.NewServiceManager("MyBackgroundService", "My Background Service", "This service runs in the background")
    if err != nil {
        log.Fatal(err)
    }

    if !daemon.IsDaemon() {
        fmt.Println("Starting service...")
        err = sm.Control("start")
        if err != nil {
            log.Fatal(err)
        }
        return
    }

    // 服务主逻辑
    go func() {
        for {
            log.Println("Service is running...")
            time.Sleep(1 * time.Minute)
        }
    }()

    err = sm.Run()
    if err != nil {
        log.Fatal(err)
    }
}
```

在这个例子中，我们创建了一个简单的后台服务，它每分钟记录一次日志。服务可以作为守护进程启动，并且可以通过系统服务管理器进行控制。

## 8. 故障排除

1. **服务无法启动**：
    - 检查服务配置是否正确
    - 确保程序有足够的权限
    - 查看系统日志以获取更多信息

2. **无法停止服务**：
    - 确保 `Stop` 方法正确实现
    - 检查是否有未正确关闭的资源或协程

3. **守护进程化失败**：
    - 检查系统权限
    - 确保没有其他实例已经在运行

## 9. 结论

Daemon 服务管理器提供了一种简单而强大的方式来将 Go 应用程序转换为系统服务并进行管理。通过遵循本手册中的指导和最佳实践，你可以轻松地创建可靠的后台服务和守护进程。

如果你在使用过程中遇到任何问题或需要进一步的帮助，请查阅相关文档或寻求社区支持。