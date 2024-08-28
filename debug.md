# Debug 包使用手册

## 简介

Debug 包提供了一系列用于运行时调试和获取调用者信息的实用工具。这个包可以帮助开发人员更容易地追踪代码执行路径，识别函数调用者，并获取有关当前执行环境的详细信息。

## 安装

要使用 Debug 包，首先需要将其导入到您的 Go 项目中：

```go
import "path/to/debug"
```

确保将 "path/to/debug" 替换为实际的包路径。

## 主要功能

### 1. 获取调用者信息

#### Caller 函数

`Caller` 函数返回调用者的函数名、文件路径和行号。

使用方法：

```go
function, path, line, err := debug.Caller()
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Called by %s at %s:%d\n", function, path, line)
}
```

#### CallerWithFilter 函数

`CallerWithFilter` 函数允许您在获取调用者信息时应用过滤器。

使用方法：

```go
filters := []string{"somepackage", "anotherpackage"}
function, path, line, err := debug.CallerWithFilter(filters)
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Called by %s at %s:%d\n", function, path, line)
}
```

### 2. 获取包和函数信息

#### CallerPackage 函数

返回调用者的包名。

使用方法：

```go
packageName := debug.CallerPackage()
fmt.Printf("Called from package: %s\n", packageName)
```

#### CallerFunction 函数

返回调用者的函数名。

使用方法：

```go
functionName := debug.CallerFunction()
fmt.Printf("Called by function: %s\n", functionName)
```

### 3. 文件和路径信息

#### CallerFilePath 函数

返回调用者的文件路径。

使用方法：

```go
filePath := debug.CallerFilePath()
fmt.Printf("Called from file: %s\n", filePath)
```

#### CallerDirectory 函数

返回调用者的目录路径。

使用方法：

```go
directory := debug.CallerDirectory()
fmt.Printf("Called from directory: %s\n", directory)
```

#### CallerFileLine 和 CallerFileLineShort 函数

这两个函数返回调用者的文件路径和行号，其中 `CallerFileLineShort` 只返回文件名（不包含完整路径）。

使用方法：

```go
fileLine := debug.CallerFileLine()
fileLineShort := debug.CallerFileLineShort()
fmt.Printf("Called from: %s\n", fileLine)
fmt.Printf("Called from (short): %s\n", fileLineShort)
```

### 4. 函数信息

#### FuncPath 和 FuncName 函数

这两个函数用于获取给定函数的完整路径和名称。

使用方法：

```go
func someFunction() {}

path := debug.FuncPath(someFunction)
name := debug.FuncName(someFunction)
fmt.Printf("Function path: %s\n", path)
fmt.Printf("Function name: %s\n", name)
```

### 5. 获取 Goroutine ID

#### GoroutineId 函数

获取当前 goroutine 的 ID。

使用方法：

```go
id, err := debug.GoroutineId()
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Current goroutine ID: %d\n", id)
}
```

### 6. 堆栈跟踪

#### PrintStack 函数

打印当前 goroutine 的堆栈跟踪到标准错误输出。

使用方法：

```go
debug.PrintStack()
```

#### Stack 和 StackWithFilter 函数

返回格式化的堆栈跟踪字符串。

使用方法：

```go
stackTrace := debug.Stack()
fmt.Println(stackTrace)

filteredStackTrace := debug.StackWithFilter("somepackage")
fmt.Println(filteredStackTrace)
```

### 7. 二进制版本信息

#### BinVersion 和 BinVersionMd5 函数

获取当前运行二进制文件的版本信息。

使用方法：

```go
version, err := debug.BinVersion()
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Binary version: %s\n", version)
}

md5Version, err := debug.BinVersionMd5()
if err != nil {
    fmt.Printf("Error: %v\n", err)
} else {
    fmt.Printf("Binary MD5 version: %s\n", md5Version)
}
```

## 注意事项

1. 性能考虑：频繁调用这些函数可能会对性能产生影响，特别是在高并发环境中。请谨慎使用，并考虑在发布版本中禁用或减少使用。

2. 安全性：这些函数可能会暴露代码结构信息。在处理敏感信息时要小心，不要在生产环境中暴露过多细节。

3. 错误处理：许多函数现在返回错误，确保在使用返回值之前进行适当的错误检查。

4. 跨平台兼容性：使用 `filepath.ToSlash` 确保了文件路径的跨平台一致性。

5. 自定义过滤：可以使用 `SetStackFilterKey` 函数来自定义堆栈过滤的关键字。

## 结论

Debug 包提供了强大的工具来帮助您理解和调试 Go 程序的运行时行为。通过合理使用这些函数，您可以更容易地追踪问题，理解代码流程，并提高开发效率。记住要平衡使用这些工具和维护代码性能之间的关系，并在生产环境中谨慎使用。