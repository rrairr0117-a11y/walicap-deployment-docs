# API 接口汇总索引

## 📚 文档结构

本API文档分为三个部分，包含完整的接口请求体格式和参数说明：

1. **[第1部分 - 字幕与视频处理](./API接口完整文档-第1部分-字幕视频.md)**
2. **[第2部分 - 音频、图片与格式转换](./API接口完整文档-第2部分-音频图片格式转换.md)**
3. **[第3部分 - 高级功能与下载](./API接口完整文档-第3部分-高级功能与下载.md)**
4. **[补充说明 - 数据源与路径要求](./API接口数据源与路径要求补充说明.md)** ⚠️ **重要**
5. **[统一规范 - 标准化接口格式](./API接口统一规范.md)** 🆕 **推荐**

---

## 🔍 快速索引

### 1. 字幕处理
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 去除字幕 | `/v1/video/remove-subtitle` | POST | 自动或手动去除视频字幕 |
| 添加字幕 | `/v1/subtitle/insert-ass` | POST | 将ASS字幕烧录到视频 |
| 自动生成字幕 | `/v1/video/caption` | POST | 自动识别语音生成字幕 |
| 字幕格式转换 | `/v1/subtitle/srt-to-ass` | POST | SRT转ASS格式 |

### 2. 视频处理
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 视频裁剪 | `/v1/video/trim` | POST | 裁剪视频指定时间段 |
| 视频拼接 | `/v1/video/concatenate` | POST | 多个视频合并为一个 |
| 多视频音频合成 | `/v1/video/combine-audio-with-videos` | POST | 视频片段+音频+字幕合成 |
| 视频音频合并 | `/v1/video/merge-audio` | POST | 视频与音频轨道合并 |
| 图片音频生成视频 | `/v1/video/audio-with-images` | POST | 图片序列+音频生成视频 ✨ **支持GIF动画** |
| Ken Burns效果 | `/v1/video/effects/ken-burns` | POST | 图片动态缩放平移效果 ✨ **已优化** |
| Lo-Fi长视频 | `/v1/video/lofi` | POST | Lo-Fi风格长视频（支持多图均分+音效混音）🆕 |
| 静态幻灯片 | `/v1/video/static-slideshow` | POST | 静态图片幻灯片（支持转场）🆕 |
| 视频蒙太奇 | `/v1/video/montage` | POST | 随机视频片段拼接 |
| 添加Logo水印 | `/v1/video/logo` | POST | 添加Logo到视频 |
| 添加横幅 | `/v1/video/add-banner` | POST | 添加文字横幅 |
| 生成缩略图 | `/v1/video/thumbnail` | POST | 提取视频缩略图 |
| 获取视频信息 | `/v1/video/info` | POST | 获取视频元数据 |

> **✨ 最新改进（v2.2）**:
> - **并发下载**: 所有多图接口支持10线程并发下载，速度提升10倍
> - **GIF支持**: `audio-with-images` 和 `lofi` 支持GIF动画保持
> - **WEBP支持**: 所有图片接口支持WEBP格式
> - **自动重试**: 图片下载失败自动重试3次
> - **容错机制**: 个别图片失败不影响整体任务

### 3. 音频处理
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 音频拼接 | `/v1/audio/concatenate` | POST | 多个音频文件合并 |
| 文字转语音(短) | `/v1/audio/tts/generate` | POST | 短文本TTS(立即返回) |
| 文字转语音(长) | `/v1/audio/tts/generate-long` | POST | 长文本TTS(异步处理) |
| 查询TTS状态 | `/v1/audio/tts/status/{task_id}` | GET | 查询长文本TTS任务状态 |
| 音频转写 | `/v1/media/transcribe` | POST | 语音识别转文字 |

### 4. 图片处理
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 文字转图片 | `/v1/image/text` | POST | 文字生成图片 |
| 网页截图 | `/v1/image/screenshot/webpage` | POST | 网页截图保存 |
| 图片转视频 | `/v1/image/convert/video` | POST | 单张图片转视频 |

### 5. 格式转换
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 视频格式转换 | `/v1/media/convert` | POST | 转换视频格式 |
| 音频格式转换 | `/v1/media/convert/mp3` | POST | 提取或转换音频 |
| 获取媒体信息 | `/v1/media/metadata` | POST | 获取媒体文件详细信息 |
| 检测静音片段 | `/v1/media/silence` | POST | 检测音频静音段 |

### 6. 下载功能
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 下载视频文件（BETA） | `/v1/BETA/media/download` | POST | 下载网络媒体（当前仅提供 BETA 路径） |
| 下载视频二进制 | `/v1/video/download-binary` | POST | 获取视频二进制数据 |
| S3下载文件 | `/v1/s3/download` | POST | ⚠️ 当前未对外暴露路由（仅内部能力） |

### 7. 高级功能
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 视频分割 | `/v1/video/split` | POST | 将视频分割成多段 |
| 视频编辑并拼接 | `/v1/video/edit-and-concatenate` | POST | 编辑多个片段并拼接 |
| S3上传文件（URL/Base64） | `/v1/s3/upload` | POST | URL/Base64/二进制数据上传 🆕 |
| S3上传文件（表单） | `/v1/s3/upload-file` | POST | multipart/form-data 上传 🆕 |
| S3上传文件（本地路径） | `/v1/s3/upload-local` | POST | 本地文件路径上传（ComfyUI专用）🆕 |
| S3列出文件 | `/v1/s3/list` | POST | 按bucket+prefix列出对象key |
| 查询任务状态 | `/v1/toolkit/job/status` | POST | 查询单个任务状态 |
| 批量查询状态 | `/v1/toolkit/jobs/status` | POST | 批量查询任务状态 |
| 文本翻译 | `/v1/text/translate` | POST | 多语言文本翻译 |
| FFmpeg自定义 | `/v1/ffmpeg/compose` | POST | 执行自定义FFmpeg命令 |
| 测试接口 | `/v1/toolkit/test` | GET | API连通性测试 |
| 认证测试 | `/v1/toolkit/authenticate` | GET | 认证功能测试 |

### 8. 音画同步（特殊接口）
| 接口名称 | 路径 | 方法 | 说明 |
|---------|------|------|------|
| 音画同步 | `/v1/video/sync-audio-video` | POST | 完整的音画同步+字幕处理 |

> 详细请求体格式见：[音画同步接口完整请求体说明](./音画同步接口完整请求体说明.md)

---

## 🎯 常用场景组合

### 场景1：视频字幕替换
1. 使用 `/v1/video/remove-subtitle` 去除原字幕
2. 使用 `/v1/subtitle/insert-ass` 添加新字幕

### 场景2：视频配音
1. 使用 `/v1/audio/tts/generate` 生成配音
2. 使用 `/v1/video/merge-audio` 合并音频到视频

### 场景3：视频剪辑合成
1. 使用 `/v1/video/trim` 裁剪多个片段
2. 使用 `/v1/video/concatenate` 拼接片段
3. 使用 `/v1/video/logo` 添加水印

### 场景4：图片视频制作
1. 使用 `/v1/video/audio-with-images` 或 `/v1/video/effects/ken-burns` 生成视频
2. 使用 `/v1/video/caption` 添加字幕
3. 使用 `/v1/video/logo` 添加品牌标识

### 场景5：音频处理流程
1. 使用 `/v1/media/transcribe` 转写音频
2. 使用 `/v1/text/translate` 翻译文本
3. 使用 `/v1/audio/tts/generate` 生成新语音

---

## ⚠️ 重要说明

### 请求格式
- **Content-Type**: 必须为 `application/json`
- **编码**: UTF-8
- **注释**: JSON不支持注释，文档中的注释仅用于说明

### 参数说明
- **必填参数**: 文档中标记为"必填"的参数必须提供
- **可选参数**: 未标记必填的参数为可选
- **默认值**: 大部分可选参数都有默认值
- **范围限制**: 请严格遵守参数的取值范围

### 异步处理
- 大文件或长时间处理建议使用异步模式
- 通过 `webhook_url` 接收处理结果
- 使用任务ID查询处理状态

### 认证方式
- **Bearer Token**: 部分接口需要在Header中提供
- **x-api-key**: 部分接口需要API密钥
- **Token有效期**: 通常为24小时

### 错误处理
- 所有接口都返回标准错误格式
- 包含错误代码、消息和详细信息
- HTTP状态码遵循RESTful规范

### 性能建议
- 视频处理使用GPU加速（`use_nvidia: true`）
- 大文件使用分片上传
- 批量处理使用并行模式
- 合理设置超时时间

---

## 📞 技术支持

遇到问题时，请提供：
1. 完整的请求体
2. 错误响应信息
3. 请求ID（request_id）
4. 时间戳

---

**文档版本**: v2.2  
**最后更新**: 2024-12-25  
**API版本**: v1  
**重大更新**: 新增 Lo-Fi/静态幻灯片接口，优化图片下载性能，支持 GIF/WEBP 格式

