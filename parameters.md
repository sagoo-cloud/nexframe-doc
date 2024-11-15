# API 参数传递指南

## 1. 支持的请求方式

框架支持以下 HTTP 请求方法：
- GET
- POST
- PUT
- PATCH
- DELETE

## 2. 内容类型（Content-Type）支持

框架支持以下几种内容类型：
- `application/json`
- `application/x-www-form-urlencoded`
- `multipart/form-data`

## 3. 参数传递方式

### 3.1 GET 请求参数

支持多种格式的参数传递：

```javascript
// 基础参数
/api/example?name=test&age=18

// 数组参数
// 方式1
/api/example?ids=1&ids=2&ids=3
// 方式2
/api/example?ids[]=1&ids[]=2&ids[]=3
// 方式3
/api/example?ids[0]=1&ids[1]=2&ids[2]=3

// 对象参数
/api/example?user.name=test&user.age=18

// 嵌套对象
/api/example?user.address.city=beijing&user.address.street=test
```

### 3.2 POST/PUT/PATCH 请求参数

#### JSON 格式
```javascript
// Content-Type: application/json
{
    "name": "test",
    "age": 18,
    "addresses": [
        {
            "city": "beijing",
            "street": "test"
        }
    ],
    "metadata": {
        "key1": "value1",
        "key2": "value2"
    }
}
```

#### Form 表单格式
```javascript
// Content-Type: application/x-www-form-urlencoded
name=test&age=18&addresses[0][city]=beijing&addresses[0][street]=test
```

### 3.3 文件上传

使用 `multipart/form-data` 格式：

```javascript
// 前端示例代码
const formData = new FormData();
formData.append('file', fileObject);           // 单文件
formData.append('files[]', fileObject1);       // 多文件
formData.append('files[]', fileObject2);
formData.append('metadata', JSON.stringify({    // 附加数据
    description: 'test upload'
}));
```

文件上传限制：
- 默认单个文件大小限制：32MB
- 支持同时上传多个文件
- 支持文件类型验证
- 支持文件大小验证

### 3.4 DELETE 请求参数

DELETE 请求支持两种方式传参：

1. URL 查询参数：
```javascript
/api/example?id=123&reason=test
```

2. 请求体参数（JSON）：
```javascript
// Content-Type: application/json
{
    "id": 123,
    "reason": "test"
}
```

## 4. 特殊参数处理

### 4.1 Map 类型参数

支持多层嵌套的 Map 结构：

```javascript
// JSON 格式
{
    "settings": {
        "theme": {
            "primary": "#fff",
            "secondary": "#000"
        }
    }
}

// URL 参数格式
?settings[theme][primary]=#fff&settings[theme][secondary]=#000
```

### 4.2 数组嵌套对象

```javascript
// JSON 格式
{
    "users": [
        {
            "name": "user1",
            "age": 20
        },
        {
            "name": "user2",
            "age": 30
        }
    ]
}

// URL 参数格式
?users[0][name]=user1&users[0][age]=20&users[1][name]=user2&users[1][age]=30
```

## 5. 最佳实践建议

1. **GET 请求**：
    - 用于数据查询
    - 参数应简单明了
    - 复杂查询条件建议使用 POST

2. **POST 请求**：
    - 推荐使用 JSON 格式传递复杂数据结构
    - 文件上传使用 multipart/form-data

3. **PUT/PATCH 请求**：
    - 建议使用 JSON 格式
    - 明确指定需要更新的字段

4. **DELETE 请求**：
    - 简单删除操作使用 URL 参数
    - 复杂删除逻辑使用请求体传参

5. **文件上传**：
    - 设置适当的超时时间
    - 添加加载状态提示
    - 建议添加文件类型和大小检查