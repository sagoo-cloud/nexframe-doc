# zlog 使用指南

## 简介

`zlog` 是一个灵活而强大的 Go 应用程序日志记录包。它提供了一个简单的接口，用于在不同级别进行日志记录，支持结构化日志记录（键值对）和格式化字符串日志记录。

## 特性

- 多个日志级别：Trace、Debug、Info、Warn、Error、Fatal、Panic
- 支持结构化日志记录和格式化字符串日志记录
- 可配置的输出（标准输出或文件）
- 支持日志轮转
- 开发和生产环境的日志记录模式

## 安装

要在您的 Go 项目中使用 `zlog`，请按以下方式导入：

```go
import "path/to/zlog"
```

## 基本用法

### 获取日志记录器实例

要获取日志记录器实例，请使用 `GetLogger()` 函数：

```go
logger := zlog.GetLogger()
```

### 在不同级别记录日志

`zlog` 支持以下日志级别：

1. Trace（跟踪）
2. Debug（调试）
3. Info（信息）
4. Warn（警告）
5. Error（错误）
6. Fatal（致命）
7. Panic（恐慌）

您可以使用结构化方法和格式化方法在这些级别记录日志：

#### 结构化日志记录（键值对）

```go
logger.Debug("用户登录", "用户ID", 12345, "用户名", "张三")
logger.Info("服务器启动", "端口", 8080)
logger.Error("数据库连接失败", "错误", err.Error())
```

#### 格式化字符串日志记录

```go
logger.Debugf("用户 %s (ID: %d) 登录", "张三", 12345)
logger.Infof("服务器在端口 %d 上启动", 8080)
logger.Errorf("数据库连接失败：%v", err)
```

### 配置

您可以使用 `SetConfig` 方法配置日志记录器：

```go
logger.SetConfig(zlog.LogConfig{
    Level:   "debug",
    Pattern: "dev",
    Output:  "stdout",
    LogRotate: zlog.LogRotate{
        Filename:   "app.log",
        MaxSize:    100,
        MaxBackups: 3,
        MaxAge:     7,
        Compress:   true,
    },
})
```

配置选项：
- `Level`：设置最小日志级别（"trace"、"debug"、"info"、"warn"、"error"、"fatal"、"panic"）
- `Pattern`：设置输出模式（"dev" 表示人类可读输出，"prod" 表示 JSON 输出）
- `Output`：设置输出目标（"stdout" 或文件路径）
- `LogRotate`：配置日志轮转（仅在记录到文件时适用）

### 检查日志级别状态

您可以检查特定日志级别是否启用：

```go
if logger.IsDebugEnabled() {
    // 执行耗时的调试日志记录操作
}
```

## 最佳实践

1. 使用适当的日志级别：
    - Trace：用于非常详细的信息，通常仅在调试时有用
    - Debug：用于调试信息
    - Info：用于应用程序进度的一般信息
    - Warn：用于潜在的有害情况
    - Error：用于可能仍允许应用程序继续运行的错误事件
    - Fatal：用于将导致应用程序中止的严重错误事件
    - Panic：用于导致恐慌的意外错误

2. 尽可能使用结构化日志记录（键值对），因为它使日志更易于查询和分析。

3. 保持日志消息格式和键名的一致性。

4. 避免记录敏感信息（如密码、API 密钥）。

5. 适当使用日志级别，以控制不同环境中的详细程度。

## 结论

`zlog` 为您的 Go 应用程序提供了一个灵活而强大的日志记录解决方案。通过遵循本指南，您可以在项目中有效地实施日志记录，从而更容易调试问题和监控应用程序行为。