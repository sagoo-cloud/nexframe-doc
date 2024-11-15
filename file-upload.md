# NexFrame框架文件上传功能说明

## 1. 功能概述

NexFrame框架提供了完整的文件上传支持，包括：
- 单文件和多文件上传
- 不同类型文件的分类上传
- 文件类型验证
- 文件大小限制
- 自定义存储路径
- 文件元数据处理

## 2. 后端实现

### 2.1 API 定义

```go
// 基本的文件上传请求结构
type ContentUploadReq struct {
    g.Meta    `path:"/upload" method:"POST" summary:"上传文件" tags:"内容管理"`
    OrgKey    string                `json:"orgKey" description:"组织标识" v:"required#组织标识不能为空"`
    SpaceKey  string                `json:"spaceKey" description:"所属空间标识" v:"required#空间标识不能为空"`
    File      []meta.FileUploadMeta `json:"file" description:"上传文件" v:"required#上传文件不能为空"`
}

// 上传响应结构
type ContentUploadRes struct {
    Files []FileInfo `json:"files"` // 返回上传后的文件信息
}

// 文件信息结构
type FileInfo struct {
    FileName string `json:"fileName"` // 文件名
    Path     string `json:"path"`     // 文件路径
    Size     int64  `json:"size"`     // 文件大小
    Type     string `json:"type"`     // 文件类型
}
```

### 2.2 控制器实现

```go
// 控制器定义
type ContentController struct {
    F *nf.APIFramework
    Service *services.ContentService
}

// 文件上传处理方法
func (c *ContentController) Upload(ctx context.Context, req *ContentUploadReq) (res *ContentUploadRes, err error) {
    res = &ContentUploadRes{
        Files: make([]FileInfo, 0, len(req.File)),
    }

    for _, fileMeta := range req.File {
        // 文件验证
        if err := validateFile(fileMeta); err != nil {
            return nil, err
        }

        // 生成存储路径
        savePath := filepath.Join("uploads", req.OrgKey, req.SpaceKey, fileMeta.FileName)

        // 保存文件
        if err := fileMeta.SaveTo(savePath); err != nil {
            return nil, fmt.Errorf("保存文件失败: %w", err)
        }

        // 添加文件信息到响应
        res.Files = append(res.Files, FileInfo{
            FileName: fileMeta.FileName,
            Path:     savePath,
            Size:     fileMeta.Size,
            Type:     fileMeta.ContentType,
        })
    }

    return res, nil
}

// 文件验证函数
func validateFile(file meta.FileUploadMeta) error {
    // 检查文件大小
    if file.Size > maxFileSize {
        return fmt.Errorf("文件 %s 超过大小限制", file.FileName)
    }

    // 检查文件类型
    if !isAllowedType(file.ContentType) {
        return fmt.Errorf("不支持的文件类型：%s", file.ContentType)
    }

    return nil
}
```

### 2.3 多类型文件上传

```go
// 多类型文件上传请求
type MultiTypeUploadReq struct {
    g.Meta     `path:"/upload/multi" method:"POST" summary:"多类型文件上传" tags:"内容管理"`
    OrgKey     string                `json:"orgKey" v:"required#组织标识不能为空"`
    SpaceKey   string                `json:"spaceKey" v:"required#空间标识不能为空"`
    Images     []meta.FileUploadMeta `json:"images" description:"图片文件"`
    Documents  []meta.FileUploadMeta `json:"documents" description:"文档文件"`
    Videos     []meta.FileUploadMeta `json:"videos" description:"视频文件"`
}

// 处理方法
func (c *ContentController) MultiTypeUpload(ctx context.Context, req *MultiTypeUploadReq) (*ContentUploadRes, error) {
    res := &ContentUploadRes{
        Files: make([]FileInfo, 0),
    }

    // 处理图片
    for _, img := range req.Images {
        if !isImageFile(img.ContentType) {
            return nil, fmt.Errorf("非图片文件：%s", img.FileName)
        }
        // 处理图片上传...
    }

    // 处理文档
    for _, doc := range req.Documents {
        if !isDocumentFile(doc.ContentType) {
            return nil, fmt.Errorf("非文档文件：%s", doc.FileName)
        }
        // 处理文档上传...
    }

    // 处理视频
    for _, video := range req.Videos {
        if !isVideoFile(video.ContentType) {
            return nil, fmt.Errorf("非视频文件：%s", video.FileName)
        }
        // 处理视频上传...
    }

    return res, nil
}
```

## 3. 前端实现

### 3.1 基础HTML表单

```html
<!-- 基础上传表单 -->
<form id="uploadForm">
    <input type="text" name="orgKey" required />
    <input type="text" name="spaceKey" required />
    <input type="file" name="file" multiple required />
    <button type="submit">上传</button>
</form>

<script>
document.getElementById('uploadForm').onsubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    
    try {
        const response = await fetch('/api/upload', {
            method: 'POST',
            body: formData
        });
        const result = await response.json();
        console.log('上传成功:', result);
    } catch (error) {
        console.error('上传失败:', error);
    }
};
</script>
```

### 3.2 多类型文件上传

```html
<form id="multiUploadForm">
    <input type="text" name="orgKey" required />
    <input type="text" name="spaceKey" required />
    
    <div>
        <label>图片文件：</label>
        <input type="file" name="images" multiple accept="image/*" />
    </div>
    
    <div>
        <label>文档文件：</label>
        <input type="file" name="documents" multiple accept=".pdf,.doc,.docx" />
    </div>
    
    <div>
        <label>视频文件：</label>
        <input type="file" name="videos" multiple accept="video/*" />
    </div>
    
    <button type="submit">上传</button>
</form>

<script>
document.getElementById('multiUploadForm').onsubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    
    try {
        const response = await fetch('/api/upload/multi', {
            method: 'POST',
            body: formData
        });
        const result = await response.json();
        console.log('上传成功:', result);
    } catch (error) {
        console.error('上传失败:', error);
    }
};
</script>
```

### 3.3 使用 Vue 实现

```vue
<template>
  <div class="upload-component">
    <div class="form-inputs">
      <input v-model="form.orgKey" placeholder="组织标识" />
      <input v-model="form.spaceKey" placeholder="空间标识" />
    </div>
    
    <div class="upload-area"
         @drop.prevent="handleDrop"
         @dragover.prevent="dragover = true"
         @dragleave.prevent="dragover = false"
         :class="{ 'dragover': dragover }">
      
      <div class="upload-prompt">
        <span v-if="files.length">已选择 {{ files.length }} 个文件</span>
        <span v-else>拖拽文件到此处或点击选择文件</span>
      </div>
      
      <input ref="fileInput"
             type="file"
             multiple
             @change="handleFileChange"
             style="display: none" />
             
      <button @click="$refs.fileInput.click()"
              type="button">
        选择文件
      </button>
    </div>
    
    <div v-if="files.length" class="file-list">
      <div v-for="(file, index) in files"
           :key="index"
           class="file-item">
        <span class="file-name">{{ file.name }}</span>
        <span class="file-size">{{ formatSize(file.size) }}</span>
        <button @click="removeFile(index)"
                type="button"
                class="remove-btn">
          删除
        </button>
      </div>
    </div>
    
    <button @click="handleUpload"
            :disabled="!canUpload || uploading"
            class="upload-btn">
      {{ uploading ? '上传中...' : '开始上传' }}
    </button>
    
    <div v-if="progress !== null" class="progress">
      上传进度: {{ progress }}%
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        orgKey: '',
        spaceKey: ''
      },
      files: [],
      dragover: false,
      uploading: false,
      progress: null
    }
  },
  
  computed: {
    canUpload() {
      return this.files.length > 0 && 
             this.form.orgKey && 
             this.form.spaceKey && 
             !this.uploading;
    }
  },
  
  methods: {
    handleDrop(e) {
      this.dragover = false;
      this.addFiles(e.dataTransfer.files);
    },
    
    handleFileChange(e) {
      this.addFiles(e.target.files);
    },
    
    addFiles(fileList) {
      this.files.push(...Array.from(fileList));
    },
    
    removeFile(index) {
      this.files.splice(index, 1);
    },
    
    formatSize(bytes) {
      const units = ['B', 'KB', 'MB', 'GB'];
      let size = bytes;
      let unitIndex = 0;
      
      while (size >= 1024 && unitIndex < units.length - 1) {
        size /= 1024;
        unitIndex++;
      }
      
      return `${size.toFixed(2)} ${units[unitIndex]}`;
    },
    
    async handleUpload() {
      if (!this.canUpload) return;
      
      const formData = new FormData();
      formData.append('orgKey', this.form.orgKey);
      formData.append('spaceKey', this.form.spaceKey);
      
      this.files.forEach(file => {
        formData.append('file', file);
      });
      
      this.uploading = true;
      this.progress = 0;
      
      try {
        const response = await fetch('/api/upload', {
          method: 'POST',
          body: formData,
          onUploadProgress: (progressEvent) => {
            this.progress = Math.round(
              (progressEvent.loaded * 100) / progressEvent.total
            );
          }
        });
        
        const result = await response.json();
        if (result.code === 0) {
          this.$emit('upload-success', result);
          this.files = [];
          this.$message.success('上传成功');
        } else {
          throw new Error(result.message);
        }
      } catch (error) {
        this.$emit('upload-error', error);
        this.$message.error('上传失败：' + error.message);
      } finally {
        this.uploading = false;
        this.progress = null;
      }
    }
  }
}
</script>

<style scoped>
.upload-area {
  border: 2px dashed #ccc;
  padding: 20px;
  text-align: center;
  cursor: pointer;
}

.upload-area.dragover {
  border-color: #409eff;
  background: #ecf5ff;
}

.file-list {
  margin-top: 10px;
}

.file-item {
  display: flex;
  align-items: center;
  padding: 5px;
  margin: 5px 0;
  background: #f5f5f5;
}

.file-name {
  flex: 1;
}

.file-size {
  margin: 0 10px;
  color: #666;
}

.progress {
  margin-top: 10px;
  text-align: center;
}

.upload-btn {
  margin-top: 10px;
  width: 100%;
  padding: 10px;
}

.upload-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>
```

## 4. 注意事项

### 4.1 后端注意事项

1. 文件大小限制
```go
// 在配置文件中设置
const (
    MaxFileSize    = 10 << 20  // 单个文件最大10MB
    MaxTotalSize   = 50 << 20  // 总上传大小50MB
)
```

2. 文件类型验证
```go
// 允许的文件类型
var AllowedTypes = map[string][]string{
    "images": {
        "image/jpeg",
        "image/png",
        "image/gif",
    },
    "documents": {
        "application/pdf",
        "application/msword",
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
    },
    "videos": {
        "video/mp4",
        "video/quicktime",
    },
}
```

3. 安全存储路径
```go
// 安全的文件路径处理
safePath := filepath.Join(baseDir, filepath.Clean(filename))
if !strings.HasPrefix(safePath, baseDir) {
    return errors.New("非法的文件路径")
}
```

### 4.2 前端注意事项

1. 文件类型限制
```html
<input type="file" accept="image/*,.pdf,.doc,.docx" />
```

2. 文件大小检查
```javascript
function validateFile(file) {
    const maxSize = 10 * 1024 * 1024; // 10MB
    if (file.size > maxSize) {
        throw new Error(`文件大小不能超过${formatSize(maxSize)}`);
    }
}
```

3. 上传进度显示
```javascript
fetch('/api/upload', {### 4.2 前端注意事项（续）

3. 上传进度显示
```javascript
// 使用 XMLHttpRequest 实现上传进度
function uploadWithProgress(formData, onProgress) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.upload.addEventListener('progress', (event) => {
            if (event.lengthComputable) {
                const progress = Math.round((event.loaded * 100) / event.total);
                onProgress(progress);
            }
        });
        
        xhr.addEventListener('load', () => {
            if (xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error('上传失败'));
            }
        });
        
        xhr.addEventListener('error', () => reject(new Error('网络错误')));
        
        xhr.open('POST', '/api/upload');
        xhr.send(formData);
    });
}

// 使用示例
async function handleUpload() {
    const formData = new FormData();
    // ... 添加文件和其他数据
    
    try {
        const result = await uploadWithProgress(formData, (progress) => {
            console.log(`上传进度: ${progress}%`);
        });
        console.log('上传成功:', result);
    } catch (error) {
        console.error('上传失败:', error);
    }
}
```

4. 文件预览
```javascript
// 图片预览
function previewImage(file) {
    return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result);
        reader.readAsDataURL(file);
    });
}

// 使用示例
async function handleFileSelect(files) {
    for (const file of files) {
        if (file.type.startsWith('image/')) {
            const previewUrl = await previewImage(file);
            // 显示图片预览
            const img = document.createElement('img');
            img.src = previewUrl;
            previewContainer.appendChild(img);
        }
    }
}
```

5. 断点续传实现
```javascript
async function uploadChunks(file, chunkSize = 1024 * 1024) {
    const chunks = Math.ceil(file.size / chunkSize);
    const uploadedChunks = new Set();
    
    for (let i = 0; i < chunks; i++) {
        if (uploadedChunks.has(i)) continue;
        
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, file.size);
        const chunk = file.slice(start, end);
        
        const formData = new FormData();
        formData.append('chunk', chunk);
        formData.append('chunkIndex', i);
        formData.append('totalChunks', chunks);
        formData.append('filename', file.name);
        
        try {
            await fetch('/api/upload/chunk', {
                method: 'POST',
                body: formData
            });
            uploadedChunks.add(i);
        } catch (error) {
            console.error(`Chunk ${i} upload failed:`, error);
            // 可以稍后重试失败的块
        }
    }
}
```

## 5. 高级功能实现

### 5.1 图片处理

```go
// 图片处理选项
type ImageProcessOptions struct {
    MaxWidth  int
    MaxHeight int
    Quality   int
    Format    string
}

// 处理上传的图片
func processImage(file meta.FileUploadMeta, opts ImageProcessOptions) error {
    // 打开图片
    img, err := imaging.Open(file.FileHeader)
    if err != nil {
        return fmt.Errorf("打开图片失败: %w", err)
    }
    
    // 调整大小
    if opts.MaxWidth > 0 || opts.MaxHeight > 0 {
        img = imaging.Fit(img, opts.MaxWidth, opts.MaxHeight, imaging.Lanczos)
    }
    
    // 保存处理后的图片
    return imaging.Save(img, file.Path, imaging.JPEGQuality(opts.Quality))
}

// 在控制器中使用
func (c *ContentController) UploadImage(ctx context.Context, req *ImageUploadReq) (*ImageUploadRes, error) {
    for _, file := range req.Images {
        if err := processImage(file, ImageProcessOptions{
            MaxWidth:  1920,
            MaxHeight: 1080,
            Quality:   85,
            Format:    "jpeg",
        }); err != nil {
            return nil, err
        }
    }
    return &ImageUploadRes{}, nil
}
```

### 5.2 大文件分片上传

```go
// 分片上传请求
type ChunkUploadReq struct {
    g.Meta      `path:"/upload/chunk" method:"POST"`
    File        meta.FileUploadMeta `json:"chunk"`
    ChunkIndex  int                 `json:"chunkIndex"`
    TotalChunks int                 `json:"totalChunks"`
    FileHash    string              `json:"fileHash"`
    FileName    string              `json:"fileName"`
}

// 分片上传处理
func (c *ContentController) UploadChunk(ctx context.Context, req *ChunkUploadReq) (*ChunkUploadRes, error) {
    // 创建临时目录
    tempDir := filepath.Join("uploads", "temp", req.FileHash)
    if err := os.MkdirAll(tempDir, 0755); err != nil {
        return nil, err
    }
    
    // 保存分片
    chunkPath := filepath.Join(tempDir, fmt.Sprintf("%d", req.ChunkIndex))
    if err := req.File.SaveTo(chunkPath); err != nil {
        return nil, err
    }
    
    // 检查是否所有分片都已上传
    if isAllChunksUploaded(tempDir, req.TotalChunks) {
        // 合并文件
        finalPath := filepath.Join("uploads", req.FileName)
        if err := mergeChunks(tempDir, finalPath, req.TotalChunks); err != nil {
            return nil, err
        }
        
        // 清理临时文件
        os.RemoveAll(tempDir)
    }
    
    return &ChunkUploadRes{}, nil
}

// 检查分片是否完整
func isAllChunksUploaded(dir string, total int) bool {
    for i := 0; i < total; i++ {
        chunkPath := filepath.Join(dir, fmt.Sprintf("%d", i))
        if _, err := os.Stat(chunkPath); os.IsNotExist(err) {
            return false
        }
    }
    return true
}

// 合并分片
func mergeChunks(tempDir, finalPath string, totalChunks int) error {
    outFile, err := os.Create(finalPath)
    if err != nil {
        return err
    }
    defer outFile.Close()
    
    for i := 0; i < totalChunks; i++ {
        chunkPath := filepath.Join(tempDir, fmt.Sprintf("%d", i))
        chunkData, err := os.ReadFile(chunkPath)
        if err != nil {
            return err
        }
        
        if _, err := outFile.Write(chunkData); err != nil {
            return err
        }
    }
    
    return nil
}
```

### 5.3 文件秒传实现

```go
// 检查文件是否存在的请求
type CheckFileReq struct {
    g.Meta   `path:"/check/file" method:"POST"`
    FileHash string `json:"fileHash"`
    FileName string `json:"fileName"`
}

// 检查文件是否已存在
func (c *ContentController) CheckFile(ctx context.Context, req *CheckFileReq) (*CheckFileRes, error) {
    // 查找文件记录
    file, err := c.Service.FindFileByHash(ctx, req.FileHash)
    if err != nil {
        return nil, err
    }
    
    if file != nil {
        // 文件已存在，直接返回文件信息
        return &CheckFileRes{
            Exists: true,
            Path:   file.Path,
        }, nil
    }
    
    return &CheckFileRes{Exists: false}, nil
}

// 前端实现秒传
async function uploadFile(file) {
    // 计算文件哈希
    const hash = await calculateFileHash(file);
    
    // 检查文件是否存在
    const checkResult = await fetch('/api/check/file', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({
            fileHash: hash,
            fileName: file.name
        })
    }).then(r => r.json());
    
    if (checkResult.exists) {
        // 文件已存在，直接使用
        console.log('文件秒传成功');
        return checkResult;
    }
    
    // 文件不存在，执行常规上传
    return normalUpload(file, hash);
}

// 计算文件哈希
async function calculateFileHash(file) {
    const chunkSize = 2 * 1024 * 1024; // 2MB chunks
    const chunks = Math.ceil(file.size / chunkSize);
    const spark = new SparkMD5.ArrayBuffer();
    
    for (let i = 0; i < chunks; i++) {
        const chunk = await readChunk(file, i * chunkSize, chunkSize);
        spark.append(chunk);
    }
    
    return spark.end();
}

function readChunk(file, start, size) {
    return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result);
        const chunk = file.slice(start, start + size);
        reader.readAsArrayBuffer(chunk);
    });
}
```

## 6. 问题排查和最佳实践

### 6.1 常见问题及解决方案

1. 上传失败
```
- 检查文件大小是否超限
- 验证文件类型是否允许
- 确认存储路径权限
- 查看服务器错误日志
```

2. 上传超时
```
- 调整服务器超时设置
- 考虑使用分片上传
- 检查网络状况
```

3. 内存使用过高
```
- 调整文件缓冲区大小
- 使用流式处理
- 启用分片上传
```

### 6.2 性能优化建议

1. 服务器端优化
```go
// 1. 使用缓冲区处理大文件
func copyWithBuffer(dst io.Writer, src io.Reader) error {
    buf := make([]byte, 32*1024)
    _, err := io.CopyBuffer(dst, src, buf)
    return err
}

// 2. 并行处理多个文件
func processFilesParallel(files []meta.FileUploadMeta) error {
    var wg sync.WaitGroup
    errors := make(chan error, len(files))
    
    for _, file := range files {
        wg.Add(1)
        go func(f meta.FileUploadMeta) {
            defer wg.Done()
            if err := processFile(f); err != nil {
                errors <- err
            }
        }(file)
    }
    
    wg.Wait()
    close(errors)
    
    if len(errors) > 0 {
        return <-errors
    }
    return nil
}

// 3. 使用临时文件
func handleLargeFile(file meta.FileUploadMeta) error {
    tempFile, err := os.CreateTemp("", "upload-*")
    if err != nil {
        return err
    }
    defer os.Remove(tempFile.Name())
    
    // 处理文件...
    
    return nil
}
```

2. 前端优化
```javascript
// 1. 压缩图片
async function compressImage(file) {
    const options = {
        maxSizeMB: 1,
        maxWidthOrHeight: 1920,
        useWebWorker: true
    };
    try {
        return await imageCompression(file, options);
    } catch (error) {
        console.error('压缩失败:', error);
        return file;
    }
}

// 2. 并行上传
async function uploadFiles(files) {
    const promises = Array.from(files).map(file => 
        uploadSingle(file)
    );
    return Promise.all(promises);
}

// 3. 失败重试
async function uploadWithRetry(file, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await uploadFile(file);
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            await new Promise(r => setTimeout(r, 1000 * (i + 1)));
        }
    }
}
```

### 6.3 安全性建议

1. 文件类型验证
```go
func validateFileType(file meta.FileUploadMeta) error {
    // 读取文件头
    buffer := make([]byte, 512)
    source, err := file.GetFile()
    if err != nil {
        return err
    }
    defer source.Close()
    
    n, err := source.Read(buffer)
    if err != nil && err != io.EOF {
        return err
    }
    
    // 检查实际文件类型
    contentType := http.DetectContentType(buffer[:n])
    if !isAllowedType(contentType) {
        return errors.New("不允许的文件类型")
    }
    
    return nil
}
```

2. 路径安全检查
```go
func validatePath(path string) error {
    // 规范化路径
    cleanPath := filepath.Clean(path)
    
    // 检查是否包含父目录引用
    if strings.Contains(cleanPath, "..") {
        return errors.New("非法的文件路径")
    }
    
    // 检查是否在允许的目录内
    absPath, err := filepath.Abs(cleanPath)
    if err != nil {
        return err
    }
    
    if !strings.HasPrefix(absPath, config.AllowedBasePath) {
        return errors.New("路径超出允许范围")
    }
    
    return nil
}
```

3. 文件扫描
```go
func scanFile(file meta.FileUploadMeta) error {
    // 检查文件大小
    if file.Size > maxFileSize {
        return errors.New("文件过大")
    }
    
    // 检查文件类型
    if err := validateFileType(file); err != nil {
        return err
    }
    
    // 检查文件内容（可### 6.3 安全性建议（续）

3. 文件扫描（续）
```go
// 文件安全扫描配置
type FileScanConfig struct {
    // 文件类型白名单
    AllowedTypes []string
    // 文件大小限制
    MaxFileSize int64
    // 病毒扫描选项
    VirusScan bool
    // 内容检查选项
    ContentCheck bool
}

// 文件安全扫描
func scanFile(file meta.FileUploadMeta, config FileScanConfig) error {
    // 检查文件名安全性
    if !isSecureFilename(file.FileName) {
        return errors.New("不安全的文件名")
    }
    
    // 检查 MIME 类型
    if !isAllowedMimeType(file.ContentType, config.AllowedTypes) {
        return errors.New("不允许的文件类型")
    }
    
    // 文件大小检查
    if file.Size > config.MaxFileSize {
        return errors.New("文件超过大小限制")
    }
    
    // 病毒扫描
    if config.VirusScan {
        if err := scanVirus(file); err != nil {
            return fmt.Errorf("病毒扫描失败: %w", err)
        }
    }
    
    // 内容安全检查
    if config.ContentCheck {
        if err := checkContent(file); err != nil {
            return fmt.Errorf("内容检查失败: %w", err)
        }
    }
    
    return nil
}

// 文件名安全检查
func isSecureFilename(filename string) bool {
    // 移除路径分隔符
    filename = filepath.Base(filename)
    
    // 检查文件名长度
    if len(filename) > 255 {
        return false
    }
    
    // 检查文件名字符
    matched, err := regexp.MatchString(`^[a-zA-Z0-9\-_.]+$`, filename)
    if err != nil || !matched {
        return false
    }
    
    // 检查常见的危险扩展名
    dangerousExts := []string{".exe", ".dll", ".so", ".sh", ".bat"}
    ext := strings.ToLower(filepath.Ext(filename))
    for _, dangerous := range dangerousExts {
        if ext == dangerous {
            return false
        }
    }
    
    return true
}
```

## 7. 扩展功能

### 7.1 云存储支持

```go
// 云存储接口
type StorageProvider interface {
    Upload(ctx context.Context, file meta.FileUploadMeta) (string, error)
    Download(ctx context.Context, path string) (io.ReadCloser, error)
    Delete(ctx context.Context, path string) error
}

// OSS存储实现
type OSSStorage struct {
    client *oss.Client
    bucket *oss.Bucket
}

func (s *OSSStorage) Upload(ctx context.Context, file meta.FileUploadMeta) (string, error) {
    objectKey := generateObjectKey(file)
    
    // 使用临时文件避免内存占用
    tempFile, err := os.CreateTemp("", "oss-upload-*")
    if err != nil {
        return "", err
    }
    defer os.Remove(tempFile.Name())
    
    // 复制文件内容到临时文件
    src, err := file.GetFile()
    if err != nil {
        return "", err
    }
    defer src.Close()
    
    if _, err := io.Copy(tempFile, src); err != nil {
        return "", err
    }
    
    // 上传到OSS
    if err := s.bucket.PutObjectFromFile(objectKey, tempFile.Name()); err != nil {
        return "", err
    }
    
    return objectKey, nil
}
```

### 7.2 自定义存储策略

```go
// 存储策略接口
type StorageStrategy interface {
    DetermineStorage(file meta.FileUploadMeta) StorageProvider
}

// 基于文件类型的存储策略
type TypeBasedStrategy struct {
    providers map[string]StorageProvider
}

func (s *TypeBasedStrategy) DetermineStorage(file meta.FileUploadMeta) StorageProvider {
    // 根据文件类型选择存储提供者
    if strings.HasPrefix(file.ContentType, "image/") {
        return s.providers["image"]
    }
    if strings.HasPrefix(file.ContentType, "video/") {
        return s.providers["video"]
    }
    return s.providers["default"]
}

// 基于文件大小的存储策略
type SizeBasedStrategy struct {
    smallFiles  StorageProvider // < 10MB
    mediumFiles StorageProvider // 10MB - 100MB
    largeFiles  StorageProvider // > 100MB
}

func (s *SizeBasedStrategy) DetermineStorage(file meta.FileUploadMeta) StorageProvider {
    switch {
    case file.Size < 10<<20:
        return s.smallFiles
    case file.Size < 100<<20:
        return s.mediumFiles
    default:
        return s.largeFiles
    }
}
```

### 7.3 文件处理管道

```go
// 文件处理管道
type ProcessingPipeline struct {
    processors []FileProcessor
}

// 文件处理器接口
type FileProcessor interface {
    Process(file meta.FileUploadMeta) error
}

// 图片处理器
type ImageProcessor struct {
    maxWidth  int
    maxHeight int
    quality   int
}

func (p *ImageProcessor) Process(file meta.FileUploadMeta) error {
    if !strings.HasPrefix(file.ContentType, "image/") {
        return nil
    }
    
    // 处理图片...
    return nil
}

// 文档处理器
type DocumentProcessor struct {
    convertToPDF bool
    addWatermark bool
}

func (p *DocumentProcessor) Process(file meta.FileUploadMeta) error {
    if !isDocument(file.ContentType) {
        return nil
    }
    
    // 处理文档...
    return nil
}

// 使用处理管道
func processFile(file meta.FileUploadMeta) error {
    pipeline := &ProcessingPipeline{
        processors: []FileProcessor{
            &ImageProcessor{
                maxWidth:  1920,
                maxHeight: 1080,
                quality:   85,
            },
            &DocumentProcessor{
                convertToPDF: true,
                addWatermark: true,
            },
        },
    }
    
    for _, processor := range pipeline.processors {
        if err := processor.Process(file); err != nil {
            return err
        }
    }
    
    return nil
}
```

### 7.4 异步处理支持

```go
// 异步任务接口
type UploadTask interface {
    Process() error
    GetStatus() TaskStatus
    GetProgress() int
    Cancel() error
}

// 任务状态
type TaskStatus string

const (
    TaskPending   TaskStatus = "pending"
    TaskRunning   TaskStatus = "running"
    TaskComplete  TaskStatus = "complete"
    TaskFailed    TaskStatus = "failed"
    TaskCancelled TaskStatus = "cancelled"
)

// 异步上传任务
type AsyncUploadTask struct {
    ID       string
    File     meta.FileUploadMeta
    Status   TaskStatus
    Progress int
    Result   string
    Error    error
    cancel   chan struct{}
}

func (t *AsyncUploadTask) Process() error {
    t.Status = TaskRunning
    
    // 启动处理协程
    go func() {
        defer func() {
            if r := recover(); r != nil {
                t.Error = fmt.Errorf("task panic: %v", r)
                t.Status = TaskFailed
            }
        }()
        
        // 处理文件...
        for i := 0; i <= 100; i += 10 {
            select {
            case <-t.cancel:
                t.Status = TaskCancelled
                return
            default:
                t.Progress = i
                time.Sleep(time.Second)
            }
        }
        
        t.Status = TaskComplete
    }()
    
    return nil
}

// 任务管理器
type TaskManager struct {
    tasks  map[string]*AsyncUploadTask
    mu     sync.RWMutex
}

func (m *TaskManager) AddTask(file meta.FileUploadMeta) string {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    taskID := generateTaskID()
    task := &AsyncUploadTask{
        ID:     taskID,
        File:   file,
        Status: TaskPending,
        cancel: make(chan struct{}),
    }
    
    m.tasks[taskID] = task
    task.Process()
    
    return taskID
}

func (m *TaskManager) GetTask(taskID string) (*AsyncUploadTask, bool) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    task, exists := m.tasks[taskID]
    return task, exists
}
```

## 8. 监控和日志

### 8.1 性能监控

```go
// 性能指标收集
type UploadMetrics struct {
    TotalUploads    int64
    TotalSize       int64
    TotalErrors     int64
    ProcessingTime  time.Duration
    ActiveUploads   int32
}

var metrics = &UploadMetrics{}

// 监控中间件
func uploadMetricsMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        atomic.AddInt32(&metrics.ActiveUploads, 1)
        defer atomic.AddInt32(&metrics.ActiveUploads, -1)
        
        next(w, r)
        
        atomic.AddInt64(&metrics.TotalUploads, 1)
        atomic.AddInt64(&metrics.ProcessingTime, time.Since(start).Nanoseconds())
    }
}

// 指标导出
func exportMetrics() {
    prometheus.MustRegister(
        prometheus.NewCounterFunc(
            prometheus.CounterOpts{
                Name: "upload_total",
                Help: "Total number of uploads",
            },
            func() float64 {
                return float64(atomic.LoadInt64(&metrics.TotalUploads))
            },
        ),
    )
}
```

### 8.2 日志记录

```go
// 日志结构
type UploadLog struct {
    Time      time.Time
    FileName  string
    FileSize  int64
    FileType  string
    Duration  time.Duration
    Status    string
    Error     string
    ClientIP  string
}

// 日志记录器
type UploadLogger struct {
    logger *zap.Logger
}

func (l *UploadLogger) LogUpload(ctx context.Context, file meta.FileUploadMeta, duration time.Duration, err error) {
    logger := l.logger.With(
        zap.String("filename", file.FileName),
        zap.Int64("size", file.Size),
        zap.String("type", file.ContentType),
        zap.Duration("duration", duration),
    )
    
    if err != nil {
        logger.Error("Upload failed",
            zap.Error(err))
    } else {
        logger.Info("Upload successful")
    }
}
```

## 9. 示例代码和最佳实践

### 9.1 完整的上传控制器示例

```go
// 完整的文件上传控制器
type UploadController struct {
    F       *nf.APIFramework
    storage StorageProvider
    logger  *UploadLogger
    metrics *UploadMetrics
}

// 上传处理
func (c *UploadController) Upload(ctx context.Context, req *UploadRequest) (*UploadResponse, error) {
    start := time.Now()
    
    // 记录上传开始
    c.logger.LogStart(ctx, req)
    defer func() {
        c.logger.LogEnd(ctx, time.Since(start))
    }()
    
    // 验证请求
    if err := c.validateRequest(req); err != nil {
        return nil, err
    }
    
    // 处理文件
    results := make([]UploadResult, 0, len(req.Files))
    for _, file := range req.Files {
        result, err := c.processFile(ctx, file)
        if err != nil {
            return nil, err
        }
        results = append(results, result)
    }
    
    // 返回结果
    return &UploadResponse{
        Results: results,
    }, nil
}

// 文件处理
func (c *UploadController) processFile(ctx context.Context, file meta.FileUploadMeta) (UploadResult, error) {
    // 文件验证
    if err := c.validateFile(file); err != nil {
        return UploadResult{}, err
    }
    
    // 文件处理
    processed, err := c.processingPipeline.Process(file)
    if err != nil {
        return UploadResult{}, err
    }
    
    // 存储文件
    path, err := c.storage.Store(ctx, processed)
    if err != nil {
        return UploadResult{}, err
    }
    
    // 返回结果
    return UploadResult{
        FileName: file.FileName,
        Path:     path,
        Size:     file.Size,
        Type:     file.ContentType,
    }, nil
}
```

### 9.2 完整的前端组件示例

```vue
<template>
  <div class="upload-component">
    <div class="upload-header">
      <h3>文件上传</h3>
      <div class="upload-stats" v-if="stats.total > 0">
        已上传: {{ stats.successful }}/{{ stats.total }}
        失败: {{ stats.failed }}
      </div>
    </div>
    
    <div
      class="upload-dropzone"
      :class="{ 'is-dragover': isDragging }"
      @dragenter.prevent="isDragging = true"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @drop.prevent="handleDrop"
      @click="$refs.fileInput.click()"
    >
      <input
        type="file"
        ref="fileInput"
        multiple
        :accept="acceptedTypes"
        @change="handleFileSelect"
        class="hidden"
      />
      
      <div class="upload-prompt">
        <i class="upload-icon"></i>
        <p>拖拽文件到此处或点击选择文件</p>
        <p class="upload-hint">
          支持的文件类型: {{ supportedTypes.join(', ') }}
          <br>
          单文件最大: {{ formatSize(maxFileSize) }}
        </p>
      </div>
    </div>
    
    <div v-if="### 9.2 完整的前端组件示例（续）

```vue
    <div v-if="files.length" class="file-list">
      <div
        v-for="(file, index) in files"
        :key="file.id"
        class="file-item"
        :class="{ 'is-error': file.error }"
      >
        <div class="file-info">
          <span class="file-name">{{ file.name }}</span>
          <span class="file-size">{{ formatSize(file.size) }}</span>
          <span class="file-type">{{ file.type }}</span>
        </div>
        
        <div class="file-status">
          <template v-if="file.status === 'pending'">
            <button
              @click="uploadFile(file)"
              class="upload-btn"
            >
              上传
            </button>
            <button
              @click="removeFile(index)"
              class="remove-btn"
            >
              删除
            </button>
          </template>
          
          <template v-else-if="file.status === 'uploading'">
            <div class="progress-bar">
              <div
                class="progress"
                :style="{ width: file.progress + '%' }"
              ></div>
              <span class="progress-text">
                {{ file.progress }}%
              </span>
            </div>
            <button
              @click="cancelUpload(file)"
              class="cancel-btn"
            >
              取消
            </button>
          </template>
          
          <template v-else-if="file.status === 'success'">
            <span class="success-text">上传成功</span>
          </template>
          
          <template v-else-if="file.status === 'error'">
            <span class="error-text">{{ file.error }}</span>
            <button
              @click="retryUpload(file)"
              class="retry-btn"
            >
              重试
            </button>
          </template>
        </div>
      </div>
    </div>
    
    <div class="upload-actions" v-if="hasPendingFiles">
      <button
        @click="uploadAllFiles"
        :disabled="uploading"
        class="upload-all-btn"
      >
        {{ uploading ? '上传中...' : '上传全部' }}
      </button>
    </div>
    
    <div class="upload-tips">
      <h4>上传提示：</h4>
      <ul>
        <li>支持拖拽上传或点击选择文件</li>
        <li>单个文件大小不超过{{ formatSize(maxFileSize) }}</li>
        <li>支持批量上传，单次最多{{ maxFiles }}个文件</li>
        <li>上传失败的文件可以重试</li>
        <li>上传过程中可以取消</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileUploader',
  
  props: {
    // 支持的文件类型
    acceptedTypes: {
      type: String,
      default: '*/*'
    },
    // 最大文件大小（字节）
    maxFileSize: {
      type: Number,
      default: 10 * 1024 * 1024 // 10MB
    },
    // 最大文件数
    maxFiles: {
      type: Number,
      default: 10
    },
    // 自动上传
    autoUpload: {
      type: Boolean,
      default: false
    }
  },
  
  data() {
    return {
      files: [],
      isDragging: false,
      uploading: false,
      stats: {
        total: 0,
        successful: 0,
        failed: 0
      }
    }
  },
  
  computed: {
    hasPendingFiles() {
      return this.files.some(f => f.status === 'pending')
    },
    
    supportedTypes() {
      return this.acceptedTypes.split(',').map(t => t.trim())
    }
  },
  
  methods: {
    // 处理文件选择
    handleFileSelect(event) {
      const selectedFiles = Array.from(event.target.files)
      this.addFiles(selectedFiles)
      event.target.value = null // 重置input，允许选择相同文件
    },
    
    // 处理拖放
    handleDrop(event) {
      this.isDragging = false
      const droppedFiles = Array.from(event.dataTransfer.files)
      this.addFiles(droppedFiles)
    },
    
    // 添加文件
    addFiles(newFiles) {
      // 验证文件
      const validFiles = newFiles.filter(file => {
        if (file.size > this.maxFileSize) {
          this.showError(`文件 ${file.name} 超过大小限制`)
          return false
        }
        if (!this.isTypeAllowed(file.type)) {
          this.showError(`不支持的文件类型：${file.type}`)
          return false
        }
        return true
      })
      
      // 检查文件数量限制
      if (this.files.length + validFiles.length > this.maxFiles) {
        this.showError(`最多只能上传 ${this.maxFiles} 个文件`)
        return
      }
      
      // 添加文件到列表
      validFiles.forEach(file => {
        this.files.push({
          id: Date.now() + Math.random(),
          file,
          name: file.name,
          size: file.size,
          type: file.type,
          status: 'pending',
          progress: 0,
          error: null
        })
      })
      
      // 自动上传
      if (this.autoUpload) {
        this.uploadAllFiles()
      }
    },
    
    // 上传单个文件
    async uploadFile(file) {
      if (file.status === 'uploading') return
      
      file.status = 'uploading'
      file.error = null
      this.stats.total++
      
      try {
        const formData = new FormData()
        formData.append('file', file.file)
        
        const response = await fetch('/api/upload', {
          method: 'POST',
          body: formData,
          onUploadProgress: (e) => {
            file.progress = Math.round((e.loaded * 100) / e.total)
          }
        })
        
        if (!response.ok) {
          throw new Error('上传失败')
        }
        
        const result = await response.json()
        if (result.code !== 0) {
          throw new Error(result.message || '上传失败')
        }
        
        file.status = 'success'
        this.stats.successful++
        this.$emit('upload-success', file, result.data)
        
      } catch (error) {
        file.status = 'error'
        file.error = error.message
        this.stats.failed++
        this.$emit('upload-error', file, error)
      }
    },
    
    // 上传所有文件
    async uploadAllFiles() {
      this.uploading = true
      const pending = this.files.filter(f => f.status === 'pending')
      
      try {
        await Promise.all(pending.map(file => this.uploadFile(file)))
      } finally {
        this.uploading = false
      }
    },
    
    // 重试上传
    retryUpload(file) {
      file.status = 'pending'
      file.progress = 0
      file.error = null
      this.uploadFile(file)
    },
    
    // 取消上传
    cancelUpload(file) {
      // TODO: 实现取消上传逻辑
      file.status = 'pending'
      file.progress = 0
    },
    
    // 移除文件
    removeFile(index) {
      this.files.splice(index, 1)
    },
    
    // 工具方法
    isTypeAllowed(type) {
      return this.acceptedTypes === '*/*' || 
             this.supportedTypes.some(t => type.match(new RegExp(t.replace('*', '.*'))))
    },
    
    formatSize(bytes) {
      const units = ['B', 'KB', 'MB', 'GB']
      let size = bytes
      let unitIndex = 0
      
      while (size >= 1024 && unitIndex < units.length - 1) {
        size /= 1024
        unitIndex++
      }
      
      return `${size.toFixed(2)} ${units[unitIndex]}`
    },
    
    showError(message) {
      this.$emit('error', message)
      // 如果使用element-ui等UI库
      // this.$message.error(message)
    }
  }
}
</script>

<style scoped>
.upload-component {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.upload-dropzone {
  border: 2px dashed #ccc;
  border-radius: 4px;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.upload-dropzone.is-dragover {
  background-color: #f8f9fa;
  border-color: #409eff;
}

.file-list {
  margin-top: 20px;
}

.file-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  margin: 5px 0;
  background: #f8f9fa;
  border-radius: 4px;
}

.file-item.is-error {
  background: #fff2f0;
}

.progress-bar {
  width: 200px;
  height: 6px;
  background: #eee;
  border-radius: 3px;
  overflow: hidden;
}

.progress {
  height: 100%;
  background: #409eff;
  transition: width 0.3s ease;
}

/* 其他样式... */
</style>
```

## 10. 常见问题解答

### 10.1 如何处理大文件上传？

1. 分片上传
   - 前端将大文件切分成小块
   - 后端合并文件块
   - 支持断点续传

2. 使用流式处理
   - 避免一次性加载整个文件到内存
   - 使用缓冲区处理

3. 监控内存使用
   - 设置适当的文件大小限制
   - 使用临时文件存储

### 10.2 如何优化上传性能？

1. 前端优化
   - 压缩文件
   - 并发上传
   - 预览优化

2. 后端优化
   - 使用对象池
   - 异步处理
   - 合理的超时设置

3. 存储优化
   - 使用合适的存储介质
   - 实现缓存机制
   - CDN 加速

### 10.3 如何保证上传安全？

1. 文件验证
   - 类型检查
   - 大小限制
   - 内容扫描

2. 路径安全
   - 规范化路径
   - 防止目录遍历
   - 权限控制

3. 访问控制
   - 用户认证
   - 上传限制
   - 日志记录

## 结语

本手册详细介绍了 NexFrame 框架的文件上传功能的实现和使用。通过合理使用这些功能，可以构建安全、高效的文件上传服务。如果遇到问题，请查看相关章节或联系技术支持。