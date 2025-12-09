# API 接口完整文档 - 第2部分：音频、图片与格式转换

## 3. 音频处理接口

### 3.1 音频拼接
**接口路径**: `/v1/audio/concatenate`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "audio_urls": [                     // array, 必填, 音频URL列表
    "string",
    "string"
  ],
  
  // 拼接设置
  "crossfade": 0,                     // number, 交叉淡化时长(秒), 范围: 0-5, 默认: 0
  "normalize": true,                  // boolean, 是否标准化音量, 默认: true
  "gap": 0,                           // number, 音频间隔(秒), 范围: 0-10, 默认: 0
  
  // 输出设置
  "output_format": "mp3",             // string, 输出格式: "mp3", "wav", "aac", "flac"
  "bitrate": "192k",                  // string, 比特率: "128k", "192k", "256k", "320k"
  "sample_rate": 44100,               // integer, 采样率: 22050, 44100, 48000
  "channels": 2,                      // integer, 声道数: 1(单声道), 2(立体声)
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 3.2 文字转语音（TTS）- 短文本
**接口路径**: `/v1/audio/tts/generate`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "text": "string",                   // 必填, 要转换的文字内容(建议少于500字)
  
  // 语音设置
  "voice": "zh-CN-XiaoxiaoNeural",    // string, 语音模型名称
  "language": "zh-CN",                // string, 语言代码
  "gender": "female",                 // string, 性别: "male", "female", "neutral"
  
  // 语音参数
  "rate": 1.0,                        // number, 语速, 范围: 0.5-2.0, 默认: 1.0
  "pitch": 1.0,                       // number, 音调, 范围: 0.5-2.0, 默认: 1.0
  "volume": 1.0,                      // number, 音量, 范围: 0-2.0, 默认: 1.0
  
  // 情感和风格
  "style": "general",                 // string, 说话风格: "general", "chat", "newscast", "customerservice", "affectionate", "angry", "calm", "cheerful", "disgruntled", "fearful", "gentle", "lyrical", "sad", "serious"
  "style_degree": 1.0,                // number, 风格强度, 范围: 0.01-2.0, 默认: 1.0
  "role": "default",                  // string, 角色扮演: "default", "girl", "boy", "youngadult", "oldadult"
  
  // SSML支持
  "use_ssml": false,                  // boolean, 是否使用SSML标记, 默认: false
  "ssml_content": "string",           // string, SSML内容(use_ssml为true时使用)
  
  // 输出设置
  "output_format": "mp3",             // string, 输出格式: "mp3", "wav", "ogg", "opus", "webm"
  "sample_rate": 24000,               // integer, 采样率: 8000, 16000, 24000, 48000
  "bitrate": "192k",                  // string, 比特率(mp3格式): "128k", "192k", "256k"
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 3.3 长文本TTS（异步）
**接口路径**: `/v1/audio/tts/generate-long`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "text": "string",                   // 必填, 长文本内容(可以是数千字)
  
  // 语音设置（同3.2）
  "voice": "zh-CN-XiaoxiaoNeural",    // string, 语音模型
  "language": "zh-CN",                // string, 语言代码
  "rate": 1.0,                        // number, 语速, 范围: 0.5-2.0
  "pitch": 1.0,                       // number, 音调, 范围: 0.5-2.0
  "volume": 1.0,                      // number, 音量, 范围: 0-2.0
  "style": "general",                 // string, 说话风格
  "style_degree": 1.0,                // number, 风格强度
  
  // 分段设置
  "auto_split": true,                 // boolean, 是否自动分段, 默认: true
  "max_chars_per_segment": 500,       // integer, 每段最大字符数, 范围: 100-1000
  "pause_between_segments": 0.5,      // number, 段落间停顿(秒), 范围: 0-3
  "split_by_punctuation": true,       // boolean, 是否按标点符号分段, 默认: true
  
  // 批处理设置
  "parallel_processing": true,        // boolean, 是否并行处理, 默认: true
  "max_parallel_jobs": 5,             // integer, 最大并行任务数, 范围: 1-10
  
  // 输出设置
  "output_format": "mp3",             // string, 输出格式: "mp3", "wav", "ogg"
  "sample_rate": 24000,               // integer, 采样率
  "merge_segments": true,             // boolean, 是否合并所有段落, 默认: true
  
  // 回调设置
  "webhook_url": "string",            // 可选, 任务完成后的回调URL
  "notification_email": "string"      // 可选, 完成通知邮箱
}
```

### 3.4 查询TTS任务状态
**接口路径**: `/v1/audio/tts/status/{task_id}`  
**请求方法**: `GET`

#### 响应体
```json
{
  "task_id": "string",                // 任务ID
  "status": "completed",              // 状态: "pending", "processing", "completed", "failed"
  "progress": 100,                    // 进度百分比: 0-100
  "audio_url": "string",              // 音频文件URL(completed时返回)
  "duration": 120.5,                  // 音频时长(秒)
  "created_at": "2024-01-01T00:00:00Z", // 创建时间
  "completed_at": "2024-01-01T00:05:00Z", // 完成时间
  "error": null                       // 错误信息(failed时返回)
}
```

### 3.5 音频转写
**接口路径**: `/v1/media/transcribe`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "media_url": "string",              // 必填, 音频或视频文件URL
  
  // 转写设置
  "task": "transcribe",               // string, 任务类型: "transcribe"(转写), "translate"(翻译成英文)
  "language": "zh",                   // string, 源语言: "zh", "en", "ja", "ko", "auto"(自动检测)
  "model": "whisper-large",           // string, 模型: "whisper-base", "whisper-large", "funasr"
  
  // 输出设置
  "include_timestamps": true,         // boolean, 是否包含时间戳, 默认: true
  "include_srt": true,                // boolean, 是否生成SRT字幕, 默认: false
  "include_vtt": false,               // boolean, 是否生成VTT字幕, 默认: false
  "include_ass": false,               // boolean, 是否生成ASS字幕, 默认: false
  "include_word_timestamps": false,   // boolean, 是否包含词级时间戳, 默认: false
  
  // 高级设置
  "prompt": "string",                 // 可选, 提示词(帮助识别专有名词)
  "temperature": 0,                   // number, 温度参数, 范围: 0-1, 默认: 0
  "compression_ratio_threshold": 2.4, // number, 压缩比阈值
  "logprob_threshold": -1.0,         // number, 对数概率阈值
  "no_speech_threshold": 0.6,        // number, 无语音阈值
  
  // 后处理
  "remove_silence": false,            // boolean, 是否移除静音段, 默认: false
  "merge_short_segments": true,       // boolean, 是否合并短句, 默认: true
  "max_segment_length": 10,          // number, 最大句段长度(秒)
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

---

## 4. 图片处理接口

### 4.1 文字转图片
**接口路径**: `/v1/image/text`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "text": "string",                   // 必填, 要转换的文字内容
  
  // 画布设置
  "width": 1920,                      // integer, 图片宽度(像素), 范围: 100-4096, 默认: 1920
  "height": 1080,                     // integer, 图片高度(像素), 范围: 100-4096, 默认: 1080
  "background_color": "#000000",      // string, 背景颜色(十六进制), 默认: "#000000"
  "background_image": "string",       // 可选, 背景图片URL
  "background_opacity": 1.0,          // number, 背景不透明度, 范围: 0-1, 默认: 1.0
  
  // 文字样式
  "font_family": "Arial",             // string, 字体名称, 默认: "Arial"
  "font_size": 48,                    // integer, 字体大小(像素), 范围: 12-500, 默认: 48
  "color": "#FFFFFF",                 // string, 文字颜色(十六进制), 默认: "#FFFFFF"
  "bold": false,                      // boolean, 是否加粗, 默认: false
  "italic": false,                    // boolean, 是否斜体, 默认: false
  "underline": false,                 // boolean, 是否下划线, 默认: false
  
  // 文字位置
  "align": "center",                  // string, 水平对齐: "left", "center", "right", 默认: "center"
  "vertical_align": "middle",         // string, 垂直对齐: "top", "middle", "bottom", 默认: "middle"
  "padding": 20,                      // integer, 内边距(像素), 范围: 0-200, 默认: 20
  "line_height": 1.5,                 // number, 行高倍数, 范围: 0.5-3, 默认: 1.5
  
  // 文字效果
  "shadow": false,                    // boolean, 是否添加阴影, 默认: false
  "shadow_color": "#000000",          // string, 阴影颜色(shadow为true时有效)
  "shadow_blur": 5,                   // integer, 阴影模糊度(像素), 范围: 0-50
  "shadow_offset_x": 2,               // integer, 阴影X偏移(像素), 范围: -50-50
  "shadow_offset_y": 2,               // integer, 阴影Y偏移(像素), 范围: -50-50
  
  "stroke": false,                    // boolean, 是否添加描边, 默认: false
  "stroke_color": "#000000",          // string, 描边颜色(stroke为true时有效)
  "stroke_width": 2,                  // integer, 描边宽度(像素), 范围: 1-20
  
  // 输出设置
  "format": "png",                    // string, 输出格式: "png", "jpg", "webp", 默认: "png"
  "quality": 95,                      // integer, 图片质量(jpg/webp), 范围: 1-100, 默认: 95
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 4.2 网页截图
**接口路径**: `/v1/image/screenshot/webpage`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "url": "https://example.com",       // 必填, 要截图的网页URL
  
  // 视窗设置
  "width": 1920,                      // integer, 视窗宽度(像素), 范围: 320-3840, 默认: 1920
  "height": 1080,                     // integer, 视窗高度(像素), 范围: 240-2160, 默认: 1080
  "device_scale_factor": 1,           // number, 设备像素比, 范围: 1-3, 默认: 1
  
  // 截图选项
  "full_page": false,                 // boolean, 是否截取整个页面, 默认: false
  "wait_for_selector": "string",      // 可选, 等待特定元素出现(CSS选择器)
  "wait_time": 0,                     // integer, 额外等待时间(毫秒), 范围: 0-30000
  "scroll_to_bottom": false,          // boolean, 是否先滚动到底部(加载懒加载内容), 默认: false
  
  // 页面设置
  "user_agent": "string",             // 可选, 自定义User-Agent
  "viewport": {
    "width": 1920,                   // integer, 视口宽度
    "height": 1080,                  // integer, 视口高度
    "is_mobile": false,              // boolean, 是否模拟移动设备
    "has_touch": false,              // boolean, 是否支持触摸
    "is_landscape": true             // boolean, 是否横屏
  },
  
  // 认证和Cookie
  "cookies": [                        // 可选, Cookie列表
    {
      "name": "string",
      "value": "string",
      "domain": "string",
      "path": "/",
      "expires": 0,
      "httpOnly": false,
      "secure": false
    }
  ],
  "headers": {                        // 可选, 自定义请求头
    "Authorization": "Bearer token"
  },
  
  // 页面交互
  "javascript_enabled": true,         // boolean, 是否启用JavaScript, 默认: true
  "block_ads": false,                 // boolean, 是否屏蔽广告, 默认: false
  "hide_elements": ["string"],        // array, 要隐藏的元素(CSS选择器)
  "click_elements": ["string"],       // array, 要点击的元素(CSS选择器)
  
  // 输出设置
  "format": "png",                    // string, 输出格式: "png", "jpg", "webp", 默认: "png"
  "quality": 95,                      // integer, 图片质量(jpg/webp), 范围: 1-100
  "clip": {                           // 可选, 截图区域
    "x": 0,                          // integer, X坐标
    "y": 0,                          // integer, Y坐标
    "width": 800,                    // integer, 宽度
    "height": 600                    // integer, 高度
  },
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 4.3 图片转视频
**接口路径**: `/v1/image/convert/video`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "image_url": "string",              // 必填, 图片URL
  
  // 视频设置
  "duration": 5,                      // number, 视频时长(秒), 范围: 0.1-300, 默认: 5
  "fps": 30,                          // integer, 帧率, 可选值: 24, 25, 30, 60, 默认: 30
  
  // 动画效果
  "effect": "none",                   // string, 动画效果: "none", "zoom_in", "zoom_out", "pan_left", "pan_right", "fade"
  "effect_duration": 5,               // number, 效果持续时间(秒)
  "zoom_factor": 1.2,                 // number, 缩放因子(zoom效果时有效), 范围: 1.0-2.0
  
  // 输出设置
  "resolution": "1920x1080",          // string, 输出分辨率, 默认: 保持原始
  "codec": "libx264",                 // string, 视频编码器, 默认: "libx264"
  "bitrate": "5M",                    // string, 视频比特率, 默认: "5M"
  "output_format": "mp4",             // string, 输出格式: "mp4", "webm", "avi", 默认: "mp4"
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

---

## 5. 格式转换接口

### 5.1 视频格式转换
**接口路径**: `/v1/media/convert`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "media_url": "string",              // 必填, 媒体文件URL
  "output_format": "mp4",             // 必填, 输出格式: "mp4", "avi", "mov", "webm", "mkv", "flv"
  
  // 视频编码设置
  "video_codec": "libx264",           // string, 视频编码器: "libx264", "libx265", "vp9", "av1", "copy"(不重新编码)
  "video_bitrate": "5M",              // string, 视频比特率: "1M", "5M", "10M", 或具体数值
  "video_crf": 23,                    // integer, 恒定质量因子, 范围: 0-51 (越小质量越高), 默认: 23
  "video_preset": "medium",           // string, 编码预设: "ultrafast", "fast", "medium", "slow", "veryslow"
  
  // 音频编码设置
  "audio_codec": "aac",               // string, 音频编码器: "aac", "mp3", "opus", "copy"(不重新编码)
  "audio_bitrate": "192k",            // string, 音频比特率: "128k", "192k", "256k", "320k"
  "audio_sample_rate": 44100,         // integer, 采样率: 22050, 44100, 48000
  "audio_channels": 2,                // integer, 声道数: 1(单声道), 2(立体声), 6(5.1环绕)
  
  // 视频处理
  "resolution": "1920x1080",          // string, 输出分辨率, 默认: 保持原始
  "fps": 30,                          // number, 帧率, 默认: 保持原始
  "aspect_ratio": "16:9",             // string, 宽高比: "16:9", "4:3", "1:1", 默认: 保持原始
  "rotation": 0,                      // integer, 旋转角度: 0, 90, 180, 270
  
  // 高级设置
  "two_pass": false,                  // boolean, 是否使用两次编码(提高质量), 默认: false
  "hardware_acceleration": "auto",    // string, 硬件加速: "none", "auto", "nvidia", "intel", "amd"
  "remove_audio": false,              // boolean, 是否移除音频, 默认: false
  "remove_video": false,              // boolean, 是否移除视频(仅保留音频), 默认: false
  
  // 字幕处理
  "subtitle_url": "string",           // 可选, 字幕文件URL
  "subtitle_codec": "mov_text",       // string, 字幕编码: "mov_text", "srt", "ass"
  "burn_subtitle": false,             // boolean, 是否烧录字幕, 默认: false
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 5.2 音频格式转换（提取音频）
**接口路径**: `/v1/media/convert/mp3`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "media_url": "string",              // 必填, 音频或视频文件URL
  
  // 音频设置
  "output_format": "mp3",             // string, 输出格式: "mp3", "wav", "aac", "flac", "ogg", "m4a"
  "bitrate": "192k",                  // string, 比特率: "128k", "192k", "256k", "320k", "lossless"(仅flac/wav)
  "sample_rate": 44100,               // integer, 采样率: 22050, 44100, 48000, 96000
  "channels": 2,                      // integer, 声道数: 1(单声道), 2(立体声)
  
  // 音频处理
  "normalize": false,                 // boolean, 是否标准化音量, 默认: false
  "normalize_level": -3,              // number, 标准化目标电平(dB), 范围: -30到0, 默认: -3
  "remove_silence": false,            // boolean, 是否移除静音, 默认: false
  "silence_threshold": -40,           // number, 静音阈值(dB), 范围: -60到0, 默认: -40
  
  // 音频效果
  "fade_in": 0,                       // number, 淡入时长(秒), 范围: 0-10, 默认: 0
  "fade_out": 0,                      // number, 淡出时长(秒), 范围: 0-10, 默认: 0
  "speed": 1.0,                       // number, 播放速度, 范围: 0.5-2.0, 默认: 1.0
  "pitch": 0,                         // integer, 音调调整(半音), 范围: -12到12, 默认: 0
  
  // 剪辑设置
  "start_time": 0,                    // number, 开始时间(秒), 默认: 0
  "end_time": null,                   // number, 结束时间(秒), null表示到结尾
  
  // 元数据
  "metadata": {
    "title": "string",               // 可选, 标题
    "artist": "string",              // 可选, 艺术家
    "album": "string",               // 可选, 专辑
    "year": "string",                // 可选, 年份
    "genre": "string",               // 可选, 流派
    "comment": "string"              // 可选, 注释
  },
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 5.3 获取媒体信息
**接口路径**: `/v1/media/metadata`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "media_url": "string",              // 必填, 媒体文件URL
  
  // 查询选项
  "include_streams": true,            // boolean, 是否包含流信息, 默认: true
  "include_format": true,             // boolean, 是否包含格式信息, 默认: true
  "include_chapters": true,           // boolean, 是否包含章节信息, 默认: false
  "include_programs": true,           // boolean, 是否包含节目信息, 默认: false
  "include_version": false            // boolean, 是否包含版本信息, 默认: false
}
```

### 5.4 检测静音片段
**接口路径**: `/v1/media/silence`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "media_url": "string",              // 必填, 音频或视频文件URL
  
  // 检测参数
  "silence_threshold": -40,           // number, 静音阈值(分贝), 范围: -60到0, 默认: -40
  "min_silence_duration": 1.0,        // number, 最小静音时长(秒), 范围: 0.1-10, 默认: 1.0
  "noise_tolerance": 0.01,            // number, 噪音容忍度, 范围: 0-0.1, 默认: 0.01
  
  // 输出选项
  "include_non_silence": false,       // boolean, 是否包含非静音片段, 默认: false
  "merge_adjacent": true,             // boolean, 是否合并相邻静音段, 默认: true
  "merge_gap": 0.5,                  // number, 合并间隔(秒), 范围: 0-5, 默认: 0.5
  
  // 处理选项
  "auto_remove": false,               // boolean, 是否自动移除静音并输出新文件, 默认: false
  "keep_edges": true,                // boolean, 是否保留首尾静音, 默认: true
  "edge_duration": 0.5,              // number, 保留的首尾静音时长(秒), 默认: 0.5
  
  // 异步处理
  "webhook_url": "string"             // 可选, 回调URL
}
```

---

**注意事项：**
1. 所有带 `// 注释` 的内容仅用于说明，实际使用时JSON不支持注释，需要删除
2. 参数标记为"必填"的必须提供，其他为可选参数
3. 数值范围和可选值请严格遵守说明中的限制
4. webhook_url 用于异步处理，处理完成后会向该URL发送结果
