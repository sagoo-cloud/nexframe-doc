# AK/SK 鉴权

ak/sk是一种身份认证方式，常用于系统间接口调用时的身份验证，其中ak为Access Key ID，sk为Secret Access Key。客户端和服务端两者会协商保存一份相同的sk，其中sk必须保密。

## 1. 说明

当系统提供OpenAPIs功能时，为了方便第三方应用直接调用系统相关的开发接口。通常接口是通过AK/SK的方式进行鉴权。需要调用方跟据Secret Key与Access Key值计算签名。

NexFrame提供了签名验证程序。

这个签名验证程序提供了一种安全的方法来验证请求的真实性和完整性。它使用 HMAC-SHA256 算法生成和验证签名，可以有效防止请求被篡改或伪造。
主要功能：

* 生成基于 HMAC-SHA256 的签名
* 验证签名的有效性


## 2. AK/SK认证过程
客户端在调用的服务端接口时候，会带一下参数进行请求，在服务端接收到这个请求的时候，首先会根据ak去数据库里面去找到对应的sk，然后使用sk对请求内容进行加密得到一个签名，然后对比客户端传过来的签名和服务端计算的出来的签名是否一致，如果一致则代表身份认证通过，反之则不通过。

ak：标明请求方是谁，即该例子中的ak
time：请求时间，时间戳，将会被对应的sk配合加密算法进行加密，得到一个signature签名
sign：签名，使用sk配合对应的加密算法后进行加密得到的签名。
appId: 应用id，在系统中创建，用于标识应用的唯一性。这个跟据实际应用过程中，自行选择是否使用。


## 3. 函数说明

### 3.1 VerifySignature

```go
func VerifySignature(ak, sk, timeStr, sign string) bool
```

**功能**: 验证提供的签名是否有效。

**参数**:
- `ak` (string): Access Key，用于标识请求的发起者
- `sk` (string): Secret Key，用于生成签名的密钥
- `timeStr` (string): 时间戳字符串
- `sign` (string): 需要验证的签名

**返回值**:
- `bool`: 如果签名有效返回 `true`，否则返回 `false`

**说明**:
此函数首先解析时间戳，然后使用提供的参数生成一个新的签名，并将其与提供的签名进行比较。比较过程使用了constant-time比较以防止时间攻击。

### 3.2 GenerateSignature

```go
func GenerateSignature(message string, secret string) string
```

**功能**: 生成 HMAC-SHA256 签名。

**参数**:
- `message` (string): 要签名的消息
- `secret` (string): 用于生成签名的密钥

**返回值**:
- `string`: 生成的十六进制编码的签名

**说明**:
此函数使用 HMAC-SHA256 算法生成签名，并将结果转换为十六进制字符串。

## 4. 使用示例

```go
package main

import (
    "fmt"
    "time"
    "github.com/sagoo-cloud/nexframe/auth"
)

func main() {
    ak := "your-access-key"
    sk := "your-secret-key"
    
    // 生成当前时间戳
    timestamp := time.Now().Unix()
    timeStr := fmt.Sprintf("%d", timestamp)
    
    // 构造消息
    message := "ak=" + ak + "&time=" + timeStr
    
    // 生成签名
    sign := auth.GenerateSignature(message, sk)
    
    // 验证签名
    isValid := auth.VerifySignature(ak, sk, timeStr, sign)
    
    fmt.Printf("Signature is valid: %v\n", isValid)
}
```

## 5. 注意事项

1. 保护好你的 Secret Key，不要泄露给任何人。
2. 时间戳应该使用服务器时间，并且客户端和服务器的时间应该同步。
3. 考虑增加时间戳的有效期检查，以防止重放攻击。
4. 在生产环境中，应该使用 HTTPS 来传输签名和其他参数。

## 6. 常见问题

Q: 如何处理时区问题？
A: 建议使用 UTC 时间或者将时区信息包含在签名中。

Q: 签名验证失败的常见原因有哪些？
A: 常见原因包括：时间戳不同步、Secret Key 不匹配、消息被篡改、签名算法不一致等。

Q: 如何增强签名的安全性？
A: 可以考虑在消息中添加随机数（nonce），并限制每个 nonce 只能使用一次。

---

如果你有任何其他问题或需要进一步的帮助，请随时联系我们的技术支持团队。