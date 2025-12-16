# API 接口完整文档 - 第3部分：高级功能与下载

## 6. 下载接口

### 6.1 下载视频文件
**接口路径**: `/v1/BETA/media/download`  
**请求方法**: `POST`

#### 完整请求体（以当前代码实现为准）
```json
{
  "media_url": "https://example.com/media", 
  "cookie": "string (可选)",
  "cloud_upload": true,
  "format": {
    "quality": "string (可选)",
    "format_id": "string (可选)",
    "resolution": "string (可选)",
    "video_codec": "string (可选)",
    "audio_codec": "string (可选)"
  },
  "audio": {
    "extract": false,
    "format": "string (可选)",
    "quality": "string (可选)"
  },
  "thumbnails": {
    "download": false,
    "download_all": false,
    "formats": ["string"],
    "convert": false,
    "embed_in_audio": false
  },
  "subtitles": {
    "download": false,
    "languages": ["en"],
    "format": "srt",
    "cloud_upload": true
  },
  "download": {
    "max_filesize": 0,
    "rate_limit": "string (可选)",
    "retries": 0
  },
  "webhook_url": "string (可选)",
  "id": "string (可选)"
}
```

### 6.2 下载视频二进制数据
**接口路径**: `/v1/video/download-binary`  
**请求方法**: `POST`

#### 完整请求体（以当前代码实现为准）
```json
{
  "video_url": "string",
  "format": "file",
  "filename": "string (可选)"
}
```

### 6.3 S3下载文件
**接口路径**: `/v1/s3/download`  
**请求方法**: `POST`

⚠️ 说明：当前代码仓库未对外暴露该路由（`routes/v1/s3/` 仅有 `/v1/s3/upload`）。本文档保留此节仅作内部能力说明。

#### 完整请求体
```json
{
  // 必填参数
  "bucket": "string",                 // 必填, S3桶名称
  "key": "string",                    // 必填, 文件路径/键名
  
  // S3配置
  "region": "us-east-1",              // string, AWS区域, 默认: "us-east-1"
  "endpoint": "string",               // 可选, 自定义S3端点(MinIO等)
  "access_key": "string",             // 可选, 访问密钥(不提供则使用默认配置)
  "secret_key": "string",             // 可选, 密钥(不提供则使用默认配置)
  
  // 下载选项
  "expires_in": 3600,                 // integer, 预签名URL过期时间(秒), 范围: 60-86400, 默认: 3600
  "version_id": "string",             // 可选, 对象版本ID
  "response_content_type": "string",  // 可选, 强制响应Content-Type
  "response_content_disposition": "attachment", // string, 响应头Content-Disposition
  
  // 加密选项
  "sse_customer_algorithm": "AES256", // string, 客户端加密算法
  "sse_customer_key": "string",       // string, 客户端加密密钥
  "sse_customer_key_md5": "string",   // string, 客户端加密密钥MD5
  
  // 输出选项
  "generate_presigned_url": true,     // boolean, 是否生成预签名URL, 默认: true
  "direct_download": false,           // boolean, 是否直接下载到服务器, 默认: false
  "local_path": "string",             // string, 本地保存路径(direct_download为true时)
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

---

## 7. 高级功能接口

### 7.1 视频分割
**接口路径**: `/v1/video/split`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  
  // 分割方式（三选一）
  "split_mode": "segments",           // string, 分割模式: "segments", "duration", "size", "scenes"
  
  // 按段数分割
  "num_segments": 5,                  // integer, 分割段数(split_mode为segments时), 范围: 2-100
  
  // 按时长分割
  "segment_duration": 60,             // number, 每段时长(秒)(split_mode为duration时), 范围: 1-3600
  
  // 按大小分割
  "segment_size": "100M",             // string, 每段大小(split_mode为size时): "50M", "100M", "500M"
  
  // 按场景分割
  "scene_threshold": 0.3,             // number, 场景变化阈值(split_mode为scenes时), 范围: 0.1-0.9
  "min_scene_duration": 2,            // number, 最小场景时长(秒), 范围: 0.5-30
  
  // 分割选项
  "overlap": 0,                       // number, 重叠时长(秒), 范围: 0-10, 默认: 0
  "keyframe_split": true,             // boolean, 是否在关键帧分割, 默认: true
  "maintain_audio_sync": true,        // boolean, 是否保持音画同步, 默认: true
  
  // 输出选项
  "output_format": "mp4",             // string, 输出格式, 默认: 与源文件相同
  "filename_pattern": "segment_%03d", // string, 文件名模板
  "create_playlist": true,            // boolean, 是否创建播放列表, 默认: false
  "playlist_format": "m3u8",          // string, 播放列表格式: "m3u8", "mpd"
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 7.2 视频编辑并拼接
**接口路径**: `/v1/video/edit-and-concatenate`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 视频片段列表
  "video_segments": [
    {
      "video_url": "string",          // string, 必填, 视频URL
      "start_time": "00:00:10",       // string, 开始时间(HH:MM:SS或秒数)
      "end_time": "00:00:30",         // string, 结束时间(HH:MM:SS或秒数)
      "speed": 1.0,                   // number, 播放速度, 范围: 0.25-4.0, 默认: 1.0
      "reverse": false,               // boolean, 是否倒放, 默认: false
      
      // 视频效果
      "filters": {
        "brightness": 0,              // number, 亮度调整, 范围: -1到1, 默认: 0
        "contrast": 1,                // number, 对比度, 范围: 0-2, 默认: 1
        "saturation": 1,              // number, 饱和度, 范围: 0-2, 默认: 1
        "hue": 0,                     // number, 色调, 范围: -180到180, 默认: 0
        "blur": 0,                    // number, 模糊度, 范围: 0-20, 默认: 0
        "sharpen": 0                  // number, 锐化, 范围: 0-5, 默认: 0
      },
      
      // 音频设置
      "audio_volume": 1.0,            // number, 音量, 范围: 0-2, 默认: 1.0
      "audio_fade_in": 0,             // number, 音频淡入(秒), 默认: 0
      "audio_fade_out": 0,            // number, 音频淡出(秒), 默认: 0
      "mute": false,                  // boolean, 是否静音, 默认: false
      
      // 转场效果(与下一片段之间)
      "transition": {
        "type": "fade",               // string, 类型: "none", "fade", "dissolve", "wipe", "slide"
        "duration": 0.5,              // number, 时长(秒), 范围: 0-5
        "direction": "left"           // string, 方向(slide/wipe): "left", "right", "up", "down"
      }
    }
  ],
  
  // 全局设置
  "output": {
    "resolution": "1920x1080",        // string, 输出分辨率
    "fps": 30,                        // integer, 输出帧率
    "codec": "libx264",               // string, 视频编码器
    "quality": "high",                // string, 质量: "low", "medium", "high", "lossless"
    "format": "mp4",                  // string, 输出格式
    "cloud_upload": true,             // boolean, 是否上传到云端, 默认: true
    "filename": "edited_video.mp4"    // string, 输出文件名
  },
  
  // 音频轨道
  "audio_tracks": [
    {
      "url": "string",                // string, 音频URL
      "volume": 0.5,                  // number, 音量
      "start_time": 0,                // number, 开始时间(秒)
      "loop": false                   // boolean, 是否循环
    }
  ],
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 7.3 S3上传文件
**接口路径**: `/v1/s3/upload`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "file_url": "string",               // 必填, 要上传的文件URL
  "bucket": "string",                 // 必填, 目标S3桶名称
  "key": "string",                    // 必填, 目标文件路径/键名
  
  // S3配置
  "region": "us-east-1",              // string, AWS区域, 默认: "us-east-1"
  "endpoint": "string",               // 可选, 自定义S3端点(MinIO等)
  "access_key": "string",             // 可选, 访问密钥
  "secret_key": "string",             // 可选, 密钥
  
  // 上传选项
  "acl": "private",                   // string, 访问控制: "private", "public-read", "public-read-write"
  "storage_class": "STANDARD",        // string, 存储类型: "STANDARD", "REDUCED_REDUNDANCY", "GLACIER"
  "server_side_encryption": "AES256", // string, 服务端加密: "AES256", "aws:kms"
  "metadata": {                       // object, 自定义元数据
    "Content-Type": "video/mp4",
    "x-amz-meta-custom": "value"
  },
  
  // 分片上传(大文件)
  "multipart_upload": true,           // boolean, 是否使用分片上传, 默认: 自动判断
  "part_size": "5MB",                 // string, 分片大小: "5MB", "10MB", "100MB"
  "max_concurrency": 5,               // integer, 最大并发数, 范围: 1-10
  
  // 覆盖选项
  "overwrite": false,                 // boolean, 是否覆盖已存在文件, 默认: false
  "check_etag": true,                 // boolean, 是否校验ETag, 默认: true
  
  // 生命周期
  "expiration_days": 30,              // integer, 过期天数, 范围: 1-3650
  "transition_to_glacier_days": 90,   // integer, 转换到Glacier天数
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 7.3.1 S3列出文件
**接口路径**: `/v1/s3/list`  
**请求方法**: `POST`

#### 完整请求体（以当前代码实现为准）
```json
{
  "bucket": "string (可选，未提供则使用环境变量 S3_BUCKET_NAME)",
  "prefix": "string (可选)",
  "folder_prefix": "string (可选，等价于 prefix)",
  "extensions": ["mp4"],
  "limit": 1000,
  "return_urls": false,
  "recursive": true
}
```

#### 返回示例
```json
{
  "bucket": "nca-toolkit-local",
  "prefix": "uploads/",
  "keys": [
    "uploads/a.mp4",
    "uploads/b.mp4"
  ],
  "count": 2
}
```

#### 说明
- **prefix/folder_prefix**：会自动补齐末尾 `/`（如果你传的是 `uploads` 会被当作 `uploads/`）。
- **extensions**：扩展名过滤（大小写不敏感），可写 `mp4` 或 `.mp4`。
- **return_urls=true**：会额外返回 `urls` 数组（基于 `S3_ENDPOINT_URL` 拼接）。
- **recursive=false**：会尝试返回 `common_prefixes`（用于模拟“子目录”）。

### 7.4 任务状态查询
**接口路径**: `/v1/toolkit/job/status`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "job_id": "string",                 // 必填, 任务ID
  
  // 查询选项
  "include_logs": false,              // boolean, 是否包含日志, 默认: false
  "log_lines": 100,                   // integer, 日志行数, 范围: 1-1000
  "include_result": true,             // boolean, 是否包含结果, 默认: true
  "wait_for_completion": false,       // boolean, 是否等待完成, 默认: false
  "wait_timeout": 60                  // integer, 等待超时(秒), 范围: 1-300
}
```

### 7.5 批量任务状态查询
**接口路径**: `/v1/toolkit/jobs/status`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 查询条件
  "job_ids": ["string"],              // array, 任务ID列表(可选)
  "status_filter": ["pending", "processing"], // array, 状态过滤: "pending", "processing", "completed", "failed"
  "created_after": "2024-01-01T00:00:00Z", // string, 创建时间起始(ISO 8601)
  "created_before": "2024-12-31T23:59:59Z", // string, 创建时间结束(ISO 8601)
  
  // 分页选项
  "limit": 10,                        // integer, 返回数量, 范围: 1-100, 默认: 10
  "offset": 0,                        // integer, 偏移量, 默认: 0
  "sort_by": "created_at",            // string, 排序字段: "created_at", "updated_at", "status"
  "sort_order": "desc",               // string, 排序方向: "asc", "desc"
  
  // 输出选项
  "include_logs": false,              // boolean, 是否包含日志, 默认: false
  "include_result": false,            // boolean, 是否包含结果, 默认: false
  "summary_only": false               // boolean, 是否仅返回摘要, 默认: false
}
```

### 7.6 文本翻译
**接口路径**: `/v1/text/translate`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "text": "string",                   // 必填, 要翻译的文本
  "target_language": "en",            // 必填, 目标语言代码
  
  // 语言设置
  "source_language": "auto",          // string, 源语言代码, "auto"表示自动检测
  "detect_language": true,            // boolean, 是否检测语言, 默认: true
  
  // 翻译选项
  "model": "google",                  // string, 翻译模型: "google", "deepl", "baidu", "youdao"
  "formality": "default",             // string, 正式程度: "default", "formal", "informal"
  "preserve_formatting": true,        // boolean, 是否保留格式, 默认: true
  
  // 高级选项
  "glossary": {                       // object, 术语表
    "术语1": "Term1",
    "术语2": "Term2"
  },
  "ignore_tags": ["code", "pre"],     // array, 忽略的HTML标签
  "split_sentences": true,            // boolean, 是否分句翻译, 默认: true
  "context": "string",                // string, 上下文信息(帮助提高翻译质量)
  
  // 批量翻译
  "texts": ["string"],                // array, 批量文本(与text二选一)
  "parallel": true,                   // boolean, 是否并行处理, 默认: true
  
  // 输出选项
  "include_alternatives": false,      // boolean, 是否包含备选翻译, 默认: false
  "include_confidence": false,        // boolean, 是否包含置信度, 默认: false
  "format": "text",                   // string, 输出格式: "text", "html", "markdown"
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 7.7 FFmpeg自定义命令
**接口路径**: `/v1/ffmpeg/compose`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 输入文件
  "inputs": [
    {
      "url": "string",                // string, 必填, 输入文件URL
      "options": ["-ss", "00:00:10"], // array, 输入选项(FFmpeg参数)
      "label": "input0"               // string, 输入标签(用于复杂滤镜)
    }
  ],
  
  // 滤镜设置
  "filters": "scale=1920:1080,fps=30", // string, 滤镜链
  "complex_filter": "string",         // string, 复杂滤镜图
  
  // 输出选项
  "output_options": [                 // array, 输出选项(FFmpeg参数)
    "-c:v", "libx264",
    "-crf", "23",
    "-preset", "medium",
    "-c:a", "aac",
    "-b:a", "192k"
  ],
  
  // 全局选项
  "global_options": [                 // array, 全局选项(FFmpeg参数)
    "-hide_banner",
    "-loglevel", "info"
  ],
  
  // 输出配置
  "output": {
    "cloud_upload": true,             // boolean, 是否上传到云端
    "filename": "output.mp4",         // string, 输出文件名
    "format": "mp4"                   // string, 强制输出格式
  },
  
  // 安全限制
  "max_duration": 3600,               // integer, 最大处理时长(秒), 默认: 3600
  "max_file_size": "5G",              // string, 最大文件大小, 默认: "5G"
  "allowed_codecs": ["libx264", "aac"], // array, 允许的编码器列表
  
  // 硬件加速
  "hardware_acceleration": {
    "enabled": true,                  // boolean, 是否启用硬件加速
    "type": "nvidia",                 // string, 类型: "nvidia", "intel", "amd"
    "device": 0                       // integer, 设备索引
  },
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 7.8 测试接口
**接口路径**: `/v1/toolkit/test`  
**请求方法**: `GET`

#### 响应体
```json
{
  "status": "ok",
  "message": "API is working",
  "version": "1.0.0",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### 7.9 认证测试
**接口路径**: `/v1/toolkit/authenticate`  
**请求方法**: `GET`

#### 请求头
```
x-api-key: your-api-key
```

#### 响应体
```json
{
  "authenticated": true,
  "message": "Authentication successful",
  "user": {
    "id": "user123",
    "role": "admin",
    "permissions": ["read", "write", "delete"]
  },
  "token_expires_at": "2024-01-02T00:00:00Z"
}
```

---

## 通用响应格式

### 成功响应
```json
{
  "status": "success",
  "data": {
    // 接口特定的返回数据
  },
  "message": "操作成功",
  "request_id": "req_123456",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### 错误响应
```json
{
  "status": "error",
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "参数video_url不能为空",
    "details": {
      "field": "video_url",
      "reason": "required"
    }
  },
  "request_id": "req_123456",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### 异步任务响应
```json
{
  "status": "accepted",
  "data": {
    "job_id": "job_123456",
    "status": "pending",
    "estimated_time": 60,
    "queue_position": 5
  },
  "message": "任务已接受，正在处理中",
  "request_id": "req_123456"
}
```

---

## 错误代码说明

| 错误代码 | HTTP状态码 | 说明 |
|---------|-----------|------|
| INVALID_PARAMETER | 400 | 参数无效 |
| MISSING_PARAMETER | 400 | 缺少必填参数 |
| INVALID_FORMAT | 400 | 格式不支持 |
| FILE_NOT_FOUND | 404 | 文件不存在 |
| UNAUTHORIZED | 401 | 未授权 |
| FORBIDDEN | 403 | 禁止访问 |
| RATE_LIMIT_EXCEEDED | 429 | 超过速率限制 |
| INTERNAL_ERROR | 500 | 内部错误 |
| SERVICE_UNAVAILABLE | 503 | 服务不可用 |
| TIMEOUT | 504 | 处理超时 |

---

**注意事项：**
1. 所有带 `// 注释` 的内容仅用于说明，实际使用时JSON不支持注释
2. 参数标记为"必填"的必须提供，其他为可选参数
3. 数值范围和可选值请严格遵守说明中的限制
4. 大文件处理建议使用异步模式，通过webhook_url接收结果
5. 认证token有效期通常为24小时，过期需重新获取
