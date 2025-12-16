# API接口数据源与路径要求补充说明

## 📋 问题分析

经过详细排查，发现当前文档存在以下不足：

1. **数据源支持不完整** - 大部分接口只写了URL参数，但实际很多接口同时支持S3/MinIO桶
2. **文件路径要求缺失** - 没有说明素材文件在服务器上的存放路径要求
3. **输出路径配置缺失** - 没有说明输出文件的保存位置选项

---

## ✅ 已实现：本地挂载目录（local_path/local_folder）与安全限制

### 1) 本地挂载参数写法（以当前代码实现为准）

系统支持把宿主机目录挂载进容器后，通过请求体中的 `local_path` / `local_folder` 直接读取素材。

常见写法：

```json
{
  "videos": {
    "local_folder": "/data/videos"
  },
  "audio": {
    "local_path": "/data/audio/voice.mp3"
  }
}
```

### 2) `LOCAL_MEDIA_ROOT` 安全机制

如果设置了环境变量 `LOCAL_MEDIA_ROOT`：

- 所有 `local_path` / `local_folder` **必须位于该目录内**
- 超出目录会直接报错拒绝（防止任意读宿主机文件）

如果未设置 `LOCAL_MEDIA_ROOT`：保持兼容（不做目录限制）。

### 3) 已支持本地挂载的接口（抽样）

- `/v1/video/montage`
  - `videos.local_folder`
  - `audio.local_path`
- `/v1/video/combine-audio-with-videos`
  - `videos.local_folder`
  - `audio.local_path`
  - `bg_music.local_path`
- `/v1/video/effects/ken-burns`
  - `images.local_folder`
  - `audio.local_path`
  - `background_music.local_path`
- `/v1/video/lofi`
  - `image.local_path`
  - `audio.local_folder`

---

## 🔧 需要补充的内容

### 1. 支持多数据源的接口（URL + S3/MinIO）

根据实际系统能力，以下接口应该同时支持URL和S3/MinIO数据源：

#### 视频处理类接口
```json
// 原文档只有：
{
  "video_url": "string"  // URL方式
}

// 应该补充为：
{
  // 方式1：使用URL
  "video_url": "string",              // 视频文件URL
  
  // 方式2：使用S3/MinIO（与video_url二选一）
  "video_source": {
    "bucket": "my-bucket",           // S3/MinIO桶名称
    "key": "videos/input.mp4",       // 文件路径/键名
    "endpoint": "http://minio:9000", // 可选, MinIO端点
    "region": "us-east-1"            // 可选, AWS区域
  }
}
```

**需要补充多数据源的接口列表：**

1. **字幕处理接口**
   - `/v1/video/remove-subtitle`
   - `/v1/subtitle/insert-ass`
   - `/v1/video/caption`

2. **视频处理接口**
   - `/v1/video/trim`
   - `/v1/video/concatenate`
   - `/v1/video/combine-audio-with-videos`
   - `/v1/video/merge-audio`
   - `/v1/video/audio-with-images`
   - `/v1/video/effects/ken-burns`
   - `/v1/video/logo`
   - `/v1/video/add-banner`
   - `/v1/video/thumbnail`
   - `/v1/video/info`
   - `/v1/video/split`
   - `/v1/video/edit-and-concatenate`

3. **音频处理接口**
   - `/v1/audio/concatenate`
   - `/v1/media/transcribe`

4. **图片处理接口**
   - `/v1/image/convert/video`

5. **格式转换接口**
   - `/v1/media/convert`
   - `/v1/media/convert/mp3`
   - `/v1/media/metadata`
   - `/v1/media/silence`

---

### 2. 服务器文件路径要求

#### 输入文件路径
当使用服务器本地文件时，需要遵循以下路径规范：

```json
{
  // 本地文件路径方式
  "local_file": {
    "path": "/data/input/videos/sample.mp4",  // 绝对路径
    "relative_path": "videos/sample.mp4",     // 相对路径（相对于配置的基础目录）
    "check_exists": true                       // 是否检查文件存在
  }
}
```

**标准目录结构：**
```
/data/
├── input/          # 输入文件目录
│   ├── videos/     # 视频文件
│   ├── audio/      # 音频文件
│   ├── images/     # 图片文件
│   └── subtitles/  # 字幕文件
├── output/         # 输出文件目录
│   ├── processed/  # 处理后的文件
│   └── temp/       # 临时文件
└── cache/          # 缓存目录
```

---

### 3. 输出配置补充

所有生成文件的接口都应该支持以下输出配置：

```json
{
  // 输出配置
  "output": {
    // 输出位置选择（三选一）
    "destination": "s3",              // 输出目标: "s3", "local", "url"
    
    // S3/MinIO输出
    "s3_output": {
      "bucket": "output-bucket",      // 输出桶名称
      "key": "processed/video.mp4",   // 输出路径
      "acl": "private",               // 访问控制
      "endpoint": "http://minio:9000" // MinIO端点
    },
    
    // 本地输出
    "local_output": {
      "path": "/data/output/processed/", // 输出目录
      "filename": "output_${timestamp}.mp4", // 文件名模板
      "create_dir": true              // 自动创建目录
    },
    
    // 临时URL输出
    "url_output": {
      "expires_in": 3600,             // URL过期时间(秒)
      "direct_download": false        // 是否直接下载
    },
    
    // 通用设置
    "overwrite": false,               // 是否覆盖已存在文件
    "generate_thumbnail": true,       // 是否生成缩略图
    "return_metadata": true           // 是否返回文件元数据
  }
}
```

---

## 📝 完整的请求体示例（补充版）

### 示例1：视频裁剪 - 完整数据源选项
```json
{
  // 输入源（三选一）
  // 选项1: URL
  "video_url": "https://example.com/video.mp4",
  
  // 选项2: S3/MinIO
  "video_source": {
    "bucket": "my-videos",
    "key": "raw/video.mp4",
    "endpoint": "http://minio:9000",
    "access_key": "minioadmin",      // 可选，不提供则使用默认配置
    "secret_key": "minioadmin"       // 可选，不提供则使用默认配置
  },
  
  // 选项3: 本地文件
  "local_file": {
    "path": "/data/input/videos/sample.mp4",
    "check_exists": true
  },
  
  // 处理参数
  "start_time": "00:00:10",
  "end_time": "00:01:30",
  
  // 输出配置
  "output": {
    "destination": "s3",
    "s3_output": {
      "bucket": "processed-videos",
      "key": "trimmed/output_${timestamp}.mp4"
    },
    "overwrite": false
  },
  
  // 异步处理
  "webhook_url": "https://callback.example.com/webhook"
}
```

### 示例2：音频拼接 - 混合数据源
```json
{
  // 音频列表（支持混合数据源）
  "audio_sources": [
    {
      // 第一个音频：URL
      "type": "url",
      "url": "https://example.com/audio1.mp3"
    },
    {
      // 第二个音频：S3
      "type": "s3",
      "bucket": "audio-bucket",
      "key": "music/audio2.mp3"
    },
    {
      // 第三个音频：本地文件
      "type": "local",
      "path": "/data/input/audio/audio3.mp3"
    }
  ],
  
  // 处理参数
  "crossfade": 2,
  "normalize": true,
  
  // 输出配置
  "output": {
    "destination": "local",
    "local_output": {
      "path": "/data/output/audio/",
      "filename": "merged_${date}_${time}.mp3"
    }
  }
}
```

---

## 🚨 重要提醒

### 1. 认证配置
当使用S3/MinIO时，可以通过以下方式提供认证：

```json
{
  // 方式1：在请求中提供
  "s3_config": {
    "access_key": "your-access-key",
    "secret_key": "your-secret-key",
    "endpoint": "http://minio:9000",
    "region": "us-east-1"
  },
  
  // 方式2：使用默认配置（推荐）
  "use_default_s3_config": true
}
```

### 2. 文件大小限制
```json
{
  "limits": {
    "max_file_size": "5GB",          // 最大文件大小
    "max_duration": 7200,             // 最大处理时长(秒)
    "max_resolution": "4096x2160"    // 最大分辨率
  }
}
```

### 3. 批量处理
对于需要处理多个文件的场景：

```json
{
  "batch_mode": true,
  "batch_sources": [
    // 支持混合数据源的批量输入
  ],
  "batch_output": {
    "destination": "s3",
    "preserve_structure": true,      // 保持原始目录结构
    "prefix": "batch_${job_id}/"     // 输出前缀
  }
}
```

---

## 📊 接口数据源支持矩阵

| 接口类别 | URL | S3/MinIO | 本地文件 | 批量 |
|---------|-----|----------|---------|------|
| 字幕处理 | ✅ | ✅ | ✅ | ⚠️ |
| 视频处理 | ✅ | ✅ | ✅ | ✅ |
| 音频处理 | ✅ | ✅ | ✅ | ✅ |
| 图片处理 | ✅ | ✅ | ✅ | ✅ |
| 格式转换 | ✅ | ✅ | ✅ | ⚠️ |

**说明：**
- ✅ 完全支持
- ⚠️ 部分支持（需要特定配置）
- ❌ 不支持

---

## 🔄 建议的文档更新

### 1. 每个接口都应包含
- **输入源选项**（URL/S3/本地）
- **输出配置**（S3/本地/临时URL）
- **路径要求说明**
- **认证方式说明**

### 2. 添加通用说明章节
- 数据源配置指南
- 文件路径规范
- S3/MinIO配置说明
- 批量处理指南

### 3. 示例代码完善
- 提供每种数据源的示例
- 包含完整的输出配置
- 展示错误处理方式

---

## 📌 总结

当前API文档需要补充：

1. **所有媒体处理接口**都应支持多数据源（URL + S3/MinIO + 本地文件）
2. **输出配置**需要标准化，支持多种输出目标
3. **文件路径规范**需要明确说明
4. **批量处理能力**需要文档化

这些补充将使API文档更加完整和实用，便于开发者正确使用所有功能。
