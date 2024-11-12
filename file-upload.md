# NexFrame 框架文件上传功能说明

## 1. 功能概述

框架提供了完整的文件上传支持，包括：
- 支持单文件和多文件上传
- 文件类型验证
- 文件大小限制
- 自定义存储路径
- 文件元数据处理

## 2. 核心组件

### 2.1 FileUploadMeta 结构体
```go
// meta/file_upload_meta.go
type FileUploadMeta struct {
    FileName    string                // 文件名
    Size        int64                 // 文件大小
    ContentType string                // 文件类型
    FileHeader  *multipart.FileHeader // 文件头信息
}
```

### 2.2 请求结构体示例
```go
type ContentUploadReq struct {
    g.Meta    `path:"/upload" method:"POST" summary:"上传文件" tags:"内容管理"`
    OrgKey    string                `json:"orgKey" v:"required#组织标识不能为空"`
    SpaceKey  string                `json:"spaceKey" v:"required#空间标识不能为空"`
    File      []meta.FileUploadMeta `json:"file" v:"required#上传文件不能为空"`
}
```

## 3. 使用方式

### 3.1 前端调用
```javascript
const formData = new FormData();
formData.append('orgKey', orgKey);
formData.append('spaceKey', spaceKey);
formData.append('file', file);  // 可多次 append 实现多文件上传

fetch('/api/upload', {
    method: 'POST',
    body: formData
})
```

### 3.2 后端控制器
```go
func (c *ContentController) Upload(ctx context.Context, req *ContentUploadReq) (*ContentUploadRes, error) {
    for _, fileMeta := range req.File {
        // 文件类型检测
        if err := fileMeta.DetectContentType(); err != nil {
            return nil, err
        }
        
        // 获取文件内容
        file, err := fileMeta.GetFile()
        if err != nil {
            return nil, err
        }
        defer file.Close()
        
        // 自定义文件处理逻辑...
    }
    return &ContentUploadRes{}, nil
}
```

## 4. 处理流程

1. 请求接收：
    - 框架检测到 multipart/form-data 请求
    - 调用 handleMultipartRequest 处理文件上传

2. 文件解析：
    - 解析表单数据
    - 提取文件信息
    - 创建 FileUploadMeta 实例

3. 数据验证：
    - 验证必填字段
    - 检查文件类型
    - 验证文件大小

4. 文件处理：
    - 生成存储路径
    - 保存文件
    - 记录文件信息

## 5. 注意事项

1. 文件上传限制：
    - 默认最大文件大小为 32MB
    - 可在配置中调整限制

2. 安全考虑：
    - 文件类型限制
    - 文件大小限制
    - 存储路径安全

3. 错误处理：
    - 文件不存在
    - 类型不支持
    - 存储失败

## 6. 代码示例

### 文件类型检查
```go
allowedTypes := []string{"image/jpeg", "image/png", "application/pdf"}
isAllowed := false
for _, allowed := range allowedTypes {
    if strings.HasPrefix(fileMeta.ContentType, allowed) {
        isAllowed = true
        break
    }
}
```

### 文件保存
```go
savePath := filepath.Join("uploads", orgKey, spaceKey)
if err := os.MkdirAll(savePath, 0755); err != nil {
    return nil, err
}

fileName := fmt.Sprintf("%d_%s", time.Now().UnixNano(), fileMeta.FileName)
fullPath := filepath.Join(savePath, fileName)

dst, err := os.Create(fullPath)
if err != nil {
    return nil, err
}
defer dst.Close()

if err := fileMeta.CopyTo(dst); err != nil {
    return nil, err
}
```

## 7. 最佳实践建议

1. 始终进行文件类型验证
2. 设置适当的文件大小限制
3. 使用安全的文件命名策略
4. 实现完整的错误处理
5. 记录必要的日志信息
6. 考虑文件存储的备份策略