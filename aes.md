# AES 加密包使用手册

## 简介

本 AES 加密包提供了一种简单而安全的方式来加密和解密字符串数据。它支持使用默认密钥（从文件中读取）或自定义密钥进行操作。

## 安装

确保您的项目使用了 Go Modules，然后运行以下命令：

```
go get github.com/sagoo-cloud/nexframe
```

## 配置

### 默认密钥文件

默认情况下，密钥文件应位于执行程序所在目录的 `etc/encryption/aes` 路径下。例如，如果您的程序位于 `/usr/local/bin/myapp`，则默认密钥文件路径应为 `/usr/local/bin/etc/encryption/aes`。

### 自定义密钥文件位置

您可以通过设置环境变量 `AES_KEY_FILE` 来自定义密钥文件的位置：

```
export AES_KEY_FILE=/path/to/your/custom/aes_key_file
```

## 使用方法

### 导入包

```go
import "github.com/sagoo-cloud/nexframe/utils/crypto"
```

### 加密字符串

#### 使用默认密钥加密

```go
plaintext := "Hello, World!"
encrypted, err := crypto.EncryptString(plaintext)
if err != nil {
    // 处理错误
}
fmt.Printf("加密后的字符串: %s\n", encrypted)
```

#### 使用自定义密钥加密

```go
plaintext := "Hello, World!"
customKey := "0123456789abcdef0123456789abcdef" // 32字节的AES-256密钥
encrypted, err := crypto.EncryptString(plaintext, customKey)
if err != nil {
    // 处理错误
}
fmt.Printf("使用自定义密钥加密后的字符串: %s\n", encrypted)
```

### 解密字符串

#### 使用默认密钥解密

```go
decrypted, err := crypto.DecryptString(encrypted)
if err != nil {
    // 处理错误
}
fmt.Printf("解密后的字符串: %s\n", decrypted)
```

#### 使用自定义密钥解密

```go
decrypted, err := crypto.DecryptString(encrypted, customKey)
if err != nil {
    // 处理错误
}
fmt.Printf("使用自定义密钥解密后的字符串: %s\n", decrypted)
```

### 直接使用 AES 实例

如果您需要多次使用相同的密钥进行加密或解密，可以创建一个 AES 实例：

```go
aes, err := crypto.NewAes() // 使用默认密钥
// 或者
// aes, err := crypto.NewAes(customKey) // 使用自定义密钥
if err != nil {
    // 处理错误
}

encrypted, err := aes.Encrypt("Hello, World!")
if err != nil {
    // 处理错误
}

decrypted, err := aes.Decrypt(encrypted)
if err != nil {
    // 处理错误
}
```

## 完整示例

以下是一个完整的示例，展示了如何使用本加密包：

```go
package main

import (
    "fmt"
    "log"

    "github.com/sagoo-cloud/nexframe/utils/crypto"
)

func main() {
    // 使用默认密钥
    plaintext := "这是一个需要加密的敏感信息"
    encrypted, err := crypto.EncryptString(plaintext)
    if err != nil {
        log.Fatalf("加密失败: %v", err)
    }
    fmt.Printf("加密后: %s\n", encrypted)

    decrypted, err := crypto.DecryptString(encrypted)
    if err != nil {
        log.Fatalf("解密失败: %v", err)
    }
    fmt.Printf("解密后: %s\n", decrypted)

    // 使用自定义密钥
    customKey := "0123456789abcdef0123456789abcdef" // 32字节的AES-256密钥
    encryptedCustom, err := crypto.EncryptString(plaintext, customKey)
    if err != nil {
        log.Fatalf("使用自定义密钥加密失败: %v", err)
    }
    fmt.Printf("使用自定义密钥加密后: %s\n", encryptedCustom)

    decryptedCustom, err := crypto.DecryptString(encryptedCustom, customKey)
    if err != nil {
        log.Fatalf("使用自定义密钥解密失败: %v", err)
    }
    fmt.Printf("使用自定义密钥解密后: %s\n", decryptedCustom)
}
```

## 注意事项

1. 确保密钥文件的权限设置正确，只允许必要的用户或进程访问。
2. 在生产环境中，考虑使用更安全的方式来管理密钥，如使用密钥管理服务。
3. 加密后的字符串是经过 Hex 编码的，因此其长度会是原文的好几倍。在存储时请考虑这一点。
4. 本包使用 AES-CFB 模式进行加密。如果您需要其他加密模式，可能需要修改源代码。
5. 自定义密钥必须是 16、24 或 32 字节长（分别对应 AES-128、AES-192 或 AES-256）。

## 错误处理

本包中的所有函数都返回错误作为第二个返回值。请始终检查并适当处理这些错误。常见的错误包括：

- 无法读取密钥文件
- 密钥长度不正确
- 加密或解密过程中的内部错误

