# 数据校验功能文档

## 1. 概述

我们的框架提供了一个灵活且可扩展的数据校验系统。它允许您使用简单的标签语法定义验证规则，支持内置和自定义规则，并提供了自定义错误消息的能力。

## 2. 标签语法

验证规则使用结构体字段的 `v` 标签定义。完整的标签语法如下：

```
field@rule1|rule2:param1,param2|rule3#自定义错误消息
```

- `field`: （可选）指定字段名，如果不提供，将使用结构体字段名
- `@`: 分隔字段名和规则列表
- `|`: 分隔多个规则
- `:`: 分隔规则名和参数
- `,`: 分隔多个参数
- `#`: 分隔规则和自定义错误消息

示例：
```go
type User struct {
    Username string `v:"username@required|length:3,20#用户名长度必须在3到20个字符之间"`
    Email    string `v:"email@required|email#请输入有效的电子邮件地址"`
}
```

## 3. 内置规则

我们的框架提供了以下内置规则：

- `required`: 字段不能为空
- `length:min,max`: 字符串长度必须在指定范围内
- `regex:pattern`: 字段必须匹配指定的正则表达式
- `email`: 字段必须是有效的电子邮件地址
- `phone`: 字段必须是有效的电话号码

使用示例：

```go
type Product struct {
    Name  string `v:"required|length:2,50"`
    Price int    `v:"required|min:0"`
    SKU   string `v:"required|regex:^[A-Z]{3}\\d{3}$"`
}
```

## 4. 自定义规则

您可以通过实现 `Rule` 接口来创建自定义规则：

```go
type Rule interface {
    Name() string
    Message() string
    Run(in RunInput) error
}
```

示例：创建一个 "even" 规则来验证偶数

```go
type RuleEven struct{}

func (r RuleEven) Name() string {
    return "even"
}

func (r RuleEven) Message() string {
    return "The {field} must be an even number"
}

func (r RuleEven) Run(in RunInput) error {
    value, ok := (*in.Value).(int)
    if !ok {
        return errors.New("even rule requires an integer value")
    }
    if value%2 != 0 {
        return errors.New(in.Message)
    }
    return nil
}

// 注册自定义规则
func init() {
    builtin.Register(RuleEven{})
}
```

使用自定义规则：

```go
type EvenRequest struct {
    Number int `v:"required|even#请输入偶数"`
}
```

## 5. 错误消息处理

错误消息可以通过以下方式定义：

1. 规则的默认消息（在 `Rule.Message()` 中定义）
2. 标签中的自定义消息（使用 `#` 分隔）

当验证失败时，系统会优先使用自定义消息（如果提供），否则使用默认消息。

## 6. 解析标签值

我们使用 `ParseTagValue` 函数来解析验证标签：

```go
func ParseTagValue(tag string) (field, rule, msg string) {
    re := regexp.MustCompile(`\s*((\w+)\s*@){0,1}\s*([^#]+)\s*(#\s*(.*)){0,1}\s*`)
    match := re.FindStringSubmatch(tag)

    if len(match) > 5 {
        msg = strings.TrimSpace(match[5])
        rule = strings.TrimSpace(match[3])
        field = strings.TrimSpace(match[2])
    } else {
        fmt.Printf("Invalid validation tag value: %s\n", tag)
    }

    return
}
```

这个函数解析完整的标签字符串，提取字段名、规则和自定义错误消息。

## 7. 最佳实践

1. 为每个需要验证的字段提供明确的规则。
2. 使用自定义错误消息来提供更具体的反馈。
3. 对于复杂的验证逻辑，考虑创建自定义规则。
4. 始终为必填字段添加 `required` 规则。
5. 使用正则表达式规则时要小心，确保它们不会过于复杂或限制性太强。
6. 在单元测试中覆盖各种验证场景，包括有效和无效的输入。

## 8. 完整示例

```go
type CreateUserRequest struct {
    framework.Meta `path:"/user/create" method:"POST" summary:"创建新用户" tags:"用户管理"`
    Username       string `v:"username@required|length:3,20#用户名必须是3-20个字符"`
    Email          string `v:"email@required|email#请输入有效的电子邮件地址"`
    Phone          string `v:"phone@required|phone#请输入有效的电话号码"`
    Age            int    `v:"age@required|min:18|max:120#年龄必须在18到120岁之间"`
}
```

在这个例子中，我们定义了一个创建用户的请求结构，使用各种内置规则和自定义错误消息来验证输入数据。

通过遵循这些指导原则和使用提供的功能，您可以确保 API 请求数据的一致性和有效性，提高整体的代码质量和用户体验。