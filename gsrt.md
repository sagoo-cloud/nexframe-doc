# GSRT 包使用说明

## 1. 简介

GSRT（Go String and Regex Tools）是一个综合性的 Go 语言工具包，提供了字符串处理、正则表达式操作和单词频率分析等功能。本包含括三个主要模块：StringUtils、RegexUtils 和 WordUtils。本手册将详细介绍 GSRT 包的使用方法、主要功能和注意事项。

## 2. 安装

使用以下命令安装 GSRT 包：

```
go get github.com/your-username/gsrt
```

## 3. 模块概览

### 3.1 StringUtils

StringUtils 模块提供了一系列常用的字符串操作函数，包括子串提取、大小写转换、字符串搜索等功能。

### 3.2 RegexUtils

RegexUtils 模块提供了处理正则表达式相关操作的函数，包括正则匹配、替换，以及一些特定的字符串验证功能。

### 3.3 WordUtils

WordUtils 模块提供了单词频率分析的功能，支持高效的单词统计和排序。

## 4. StringUtils 模块

### 4.1 主要函数

#### SubString

```go
func SubString(str string, start int, end int) string
```

截取字符串的一部分。

**示例：**
```go
s := "Hello, World!"
fmt.Println(SubString(s, 0, 5))  // 输出: "Hello"
fmt.Println(SubString(s, 7, -1)) // 输出: "World!"
```

#### FirstToUpper

```go
func FirstToUpper(str string) string
```

将字符串的第一个字符转换为大写。

**示例：**
```go
fmt.Println(FirstToUpper("hello")) // 输出: "Hello"
```

#### FirstToLower

```go
func FirstToLower(str string) string
```

将字符串的第一个字符转换为小写。

**示例：**
```go
fmt.Println(FirstToLower("Hello")) // 输出: "hello"
```

#### InStringArray

```go
func InStringArray(needle string, haystack []string) bool
```

检查一个字符串是否在字符串数组中。

**示例：**
```go
arr := []string{"apple", "banana", "cherry"}
fmt.Println(InStringArray("banana", arr)) // 输出: true
fmt.Println(InStringArray("date", arr))   // 输出: false
```

#### ResolveAddress

```go
func ResolveAddress(addr []string) (string, error)
```

解析地址参数。

**示例：**
```go
addr, err := ResolveAddress([]string{"localhost", "8080"})
if err != nil {
    fmt.Println("错误:", err)
} else {
    fmt.Println("地址:", addr)
}
// 输出: 地址: localhost:8080
```

#### ReplaceIndex

```go
func ReplaceIndex(s, old, new string, n int) string
```

替换字符串中第 n 次出现的旧字符串。

**示例：**
```go
s := "The quick brown fox jumps over the lazy dog"
fmt.Println(ReplaceIndex(s, "the", "a", 1))
// 输出: "The quick brown fox jumps over a lazy dog"
```

#### UnderLineName

```go
func UnderLineName(name string) string
```

将驼峰命名转换为下划线命名。

**示例：**
```go
fmt.Println(UnderLineName("helloWorld"))  // 输出: "hello_world"
fmt.Println(UnderLineName("HelloWorld"))  // 输出: "hello_world"
```

#### HumpName

```go
func HumpName(name string) string
```

将下划线命名转换为驼峰命名。

**示例：**
```go
fmt.Println(HumpName("hello_world"))  // 输出: "HelloWorld"
```

## 5. RegexUtils 模块

### 5.1 主要函数

#### ExtractRegexMatch

```go
func ExtractRegexMatch(regex, content string, index int) (string, error)
```

从内容中提取正则表达式匹配的指定捕获组。

**示例：**
```go
content := "My email is example@example.com"
regex := `(\w+@\w+\.\w+)`
match, err := ExtractRegexMatch(regex, content, 1)
if err != nil {
    fmt.Println("错误:", err)
} else {
    fmt.Println("提取的邮箱:", match)
}
// 输出: 提取的邮箱: example@example.com
```

#### ReplaceRegexMatch

```go
func ReplaceRegexMatch(regex, newStr, content string) (string, error)
```

用新字符串替换所有正则表达式匹配项。

**示例：**
```go
content := "The quick brown fox jumps over the lazy dog"
regex := `\b\w{5}\b`
newStr := "----"
result, err := ReplaceRegexMatch(regex, newStr, content)
if err != nil {
    fmt.Println("错误:", err)
} else {
    fmt.Println("替换结果:", result)
}
// 输出: 替换结果: The ---- brown fox ---- over the lazy dog
```

#### IsFloat

```go
func IsFloat(s string) bool
```

判断字符串是否为浮点数。

**示例：**
```go
fmt.Println(IsFloat("3.14"))   // 输出: true
fmt.Println(IsFloat("-2.5"))   // 输出: true
fmt.Println(IsFloat("abc"))    // 输出: false
```

#### IsInt

```go
func IsInt(s string) bool
```

判断字符串是否为整数。

**示例：**
```go
fmt.Println(IsInt("123"))    // 输出: true
fmt.Println(IsInt("-456"))   // 输出: true
fmt.Println(IsInt("3.14"))   // 输出: false
```

#### GetRootDomain

```go
func GetRootDomain(url string) string
```

从 URL 中提取根域名。

**示例：**
```go
fmt.Println(GetRootDomain("https://www.example.com/page"))  // 输出: example.com
fmt.Println(GetRootDomain("sub.example.co.uk"))             // 输出: example.co.uk
```

## 6. WordUtils 模块

### 6.1 主要类型

#### WordRankResult

```go
type WordRankResult struct {
    Text  string  `json:"text"`
    Count int     `json:"count"`
    Rank  float32 `json:"rank"`
}
```

表示单词频率分析的结果。

#### WordRankResults

```go
type WordRankResults []WordRankResult
```

WordRankResult 的切片，实现了 sort.Interface。

### 6.2 主要函数

#### WordRank

```go
func WordRank(arrWords []string) WordRankResults
```

对输入的单词切片进行词频分析。

**示例：**
```go
words := []string{"apple", "banana", "apple", "cherry", "date", "apple"}
result := WordRank(words)
for _, item := range result {
    fmt.Printf("单词: %s, 次数: %d, 频率: %.2f\n", item.Text, item.Count, item.Rank)
}
```

输出：
```
单词: apple, 次数: 3, 频率: 0.50
单词: banana, 次数: 1, 频率: 0.17
单词: cherry, 次数: 1, 频率: 0.17
单词: date, 次数: 1, 频率: 0.17
```

#### ClearWordRankCache

```go
func ClearWordRankCache()
```

清除 WordRank 函数的缓存。

**示例：**
```go
ClearWordRankCache()
```

#### SetCacheDuration

```go
func SetCacheDuration(duration time.Duration)
```

设置缓存的过期时间。

**示例：**
```go
SetCacheDuration(10 * time.Minute)
```

## 7. 性能考虑

1. **StringUtils**：
   - `SubString` 函数对非 ASCII 字符串使用了 `[]rune` 转换，可能导致额外的内存分配。
   - `ReplaceIndex` 函数使用了 `strings.Builder` 来优化字符串拼接。

2. **RegexUtils**：
   - `ExtractRegexMatch` 和 `ReplaceRegexMatch` 函数在每次调用时都会编译正则表达式。对于频繁使用的正则表达式，考虑预编译。
   - `GetRootDomain` 函数使用了复杂的正则表达式，可能在处理大量 URL 时成为性能瓶颈。

3. **WordUtils**：
   - 使用了 `sync.Pool` 来重用 `map[string]int` 对象，减少内存分配和 GC 压力。
   - 使用 `sync.Map` 实现了线程安全的缓存机制。

## 8. 注意事项

1. 所有接受索引参数的函数都假设输入的索引是有效的。在实际使用中，应该在调用这些函数之前验证索引的有效性。

2. 正则表达式的编写需要小心，错误的正则表达式可能导致意外的结果或性能问题。

3. WordUtils 模块的缓存机制可能会占用额外的内存。在内存受限的环境中使用时需要注意。

## 9. 可能的扩展

1. 添加更多的字符串处理和验证函数。

2. 实现一个正则表达式缓存机制，以提高频繁使用相同正则表达式的性能。

3. 为 WordUtils 模块添加更多的分析功能，如词性标注、语言检测等。

4. 实现并行处理版本的函数，以提高大规模数据处理的效率。

## 

GSRT 包提供了一系列强大的字符串处理、正则表达式操作和单词频率分析工具。在使用这些函数时，请注意其性能特性和使用限制。