# API接口统一规范

## 🎯 统一的数据源格式

所有媒体处理接口都应支持以下三种数据源方式：

### 标准输入格式
```json
{
  // 方式1：URL输入（最简单）
  "input_url": "https://example.com/media.mp4",
  
  // 方式2：S3/MinIO输入（企业级）
  "input_source": {
    "type": "s3",
    "bucket": "my-bucket",
    "key": "path/to/media.mp4",
    "endpoint": "http://minio:9000",  // 可选，MinIO端点
    "region": "us-east-1",             // 可选，AWS区域
    "access_key": "minioadmin",        // 可选，不提供则用默认
    "secret_key": "minioadmin"         // 可选，不提供则用默认
  },
  
  // 方式3：服务器本地文件（高性能）
  "input_file": {
    "type": "local",
    "path": "/data/input/videos/sample.mp4",  // 绝对路径
    "check_exists": true                       // 检查文件是否存在
  }
}
```

## 🎯 统一的输出配置格式

所有生成文件的接口都应使用统一的输出配置：

### 标准输出格式
```json
{
  "output": {
    // 输出目标类型
    "destination": "s3",              // "s3" | "local" | "url"
    
    // S3/MinIO输出配置
    "s3": {
      "bucket": "output-bucket",
      "key": "processed/${timestamp}/output.mp4",  // 支持变量
      "acl": "private",                             // 访问控制
      "storage_class": "STANDARD",
      "metadata": {
        "processed_by": "ncat",
        "timestamp": "${timestamp}"
      }
    },
    
    // 本地文件输出配置
    "local": {
      "path": "/data/output/processed/",
      "filename": "output_${timestamp}.mp4",  // 支持变量
      "create_dir": true,
      "permissions": "644"
    },
    
    // 临时URL输出配置
    "url": {
      "expires_in": 3600,              // URL过期时间(秒)
      "signed": true,                  // 是否签名
      "download_name": "output.mp4"    // 下载文件名
    },
    
    // 通用配置
    "overwrite": false,                // 是否覆盖已存在文件
    "return_metadata": true,           // 是否返回文件元数据
    "generate_thumbnail": false,       // 是否生成缩略图
    "webhook_on_complete": "https://callback.example.com"  // 完成回调
  }
}
```

## 🎯 统一的文件路径规范

### 服务器目录结构
```
/data/
├── input/                    # 输入文件根目录
│   ├── videos/              # 视频文件
│   ├── audio/               # 音频文件
│   ├── images/              # 图片文件
│   ├── subtitles/           # 字幕文件
│   └── temp/                # 临时上传文件
│
├── output/                   # 输出文件根目录
│   ├── processed/           # 处理完成的文件
│   ├── thumbnails/          # 缩略图
│   ├── preview/             # 预览文件
│   └── export/              # 导出文件
│
├── cache/                    # 缓存目录
│   ├── transcoding/         # 转码缓存
│   ├── thumbnails/          # 缩略图缓存
│   └── metadata/            # 元数据缓存
│
└── workspace/                # 工作目录
    ├── projects/            # 项目文件
    └── sessions/            # 会话文件
```

## 📝 接口改进示例

### 改进前（当前文档）
```json
{
  "video_url": "https://example.com/video.mp4",
  "output_format": "mp4",
  "cloud_upload": true,
  "filename": "output.mp4"
}
```

### 改进后（统一格式）
```json
{
  // 统一的输入配置
  "input": {
    "url": "https://example.com/video.mp4"
    // 或使用 source 对象支持多种输入
  },
  
  // 统一的输出配置
  "output": {
    "destination": "s3",
    "s3": {
      "bucket": "processed-videos",
      "key": "output/${date}/video.mp4"
    },
    "format": "mp4",
    "overwrite": false
  },
  
  // 处理参数（接口特定）
  "processing": {
    // 接口特定的处理参数
  }
}
```

## 🔄 批量处理支持

### 统一的批量格式
```json
{
  "batch": {
    "enabled": true,
    "inputs": [
      {
        "id": "task-001",
        "url": "https://example.com/video1.mp4"
      },
      {
        "id": "task-002",
        "source": {
          "type": "s3",
          "bucket": "my-bucket",
          "key": "video2.mp4"
        }
      }
    ],
    "parallel": true,
    "max_concurrent": 5,
    "continue_on_error": true
  },
  
  "output": {
    "destination": "s3",
    "s3": {
      "bucket": "batch-output",
      "key_prefix": "batch/${batch_id}/"
    }
  }
}
```

## 🔑 认证配置

### 统一的认证格式
```json
{
  "auth": {
    // API认证
    "api_key": "your-api-key",        // 或在Header中
    "api_secret": "your-secret",
    
    // S3/MinIO认证
    "s3": {
      "access_key": "AKIAIOSFODNN7EXAMPLE",
      "secret_key": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
      "session_token": "temporary-token",  // 可选，临时凭证
      "region": "us-east-1",
      "endpoint": "http://minio:9000"      // MinIO端点
    },
    
    // 使用默认配置
    "use_default": true                    // 使用服务器默认配置
  }
}
```

## 📊 变量支持

所有路径和文件名支持以下变量：

| 变量 | 说明 | 示例 |
|------|------|------|
| `${timestamp}` | Unix时间戳 | 1701936000 |
| `${date}` | 日期 | 2024-12-09 |
| `${time}` | 时间 | 14-30-00 |
| `${datetime}` | 日期时间 | 2024-12-09_14-30-00 |
| `${uuid}` | UUID | 550e8400-e29b-41d4-a716 |
| `${job_id}` | 任务ID | job_abc123 |
| `${original_name}` | 原始文件名 | input_video |
| `${extension}` | 文件扩展名 | mp4 |

## ⚡ 性能优化配置

### 统一的性能配置
```json
{
  "performance": {
    "priority": "speed",               // "speed" | "quality" | "balanced"
    "hardware_acceleration": {
      "enabled": true,
      "type": "nvidia",                // "nvidia" | "intel" | "amd" | "auto"
      "device_id": 0
    },
    "threading": {
      "enabled": true,
      "max_threads": 8
    },
    "memory": {
      "max_usage": "4GB",
      "cache_size": "1GB"
    },
    "timeout": 3600,                  // 超时时间(秒)
    "retry": {
      "enabled": true,
      "max_attempts": 3,
      "backoff": "exponential"
    }
  }
}
```

## 🚀 实施建议

### 1. 逐步迁移
- **第一阶段**：新接口使用统一格式
- **第二阶段**：为旧接口提供兼容层
- **第三阶段**：完全迁移到新格式

### 2. 向后兼容
```json
{
  // 支持旧格式
  "video_url": "...",  // 自动转换为 input.url
  
  // 同时支持新格式
  "input": {
    "url": "..."
  }
}
```

### 3. 版本控制
- 使用 `/v2/` 路径表示新版本
- 保持 `/v1/` 接口不变
- 提供迁移指南

## 📋 检查清单

每个接口应该：
- ✅ 支持三种输入方式（URL/S3/本地）
- ✅ 使用统一的输出配置
- ✅ 遵循标准文件路径规范
- ✅ 支持批量处理
- ✅ 包含完整的错误处理
- ✅ 提供性能优化选项
- ✅ 支持变量替换
- ✅ 有清晰的认证说明

---

**注意**：这是建议的统一规范，实际实施需要根据系统能力逐步进行。
