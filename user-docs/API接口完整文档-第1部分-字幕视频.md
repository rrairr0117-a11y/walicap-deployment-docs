# API 接口完整文档 - 第1部分：字幕与视频处理

## 📌 接口规范说明

### 基础信息
- **基础URL**: `http://ncat:8080` (内部) 或配置的服务器地址
- **认证方式**: Bearer Token 或 x-api-key
- **Content-Type**: `application/json`
- **请求方法**: 主要为 POST，部分查询接口为 GET

---

## 1. 字幕处理接口

### 1.1 去除字幕
**接口路径**: `/v1/video/remove-subtitle`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填：视频文件URL
  "video_url": "string",              // 示例: "http://example.com/video.mp4"
  
  // 自动检测模式（推荐）
  "auto_detect": true,                // boolean, 默认: false, 是否自动检测字幕位置
  "detect_position": "bottom",        // string, 可选值: "bottom", "top", "full", 默认: "bottom"
  "sample_frames": 3,                 // integer, 采样帧数, 范围: 1-10, 默认: 3
  
  // 手动指定位置（auto_detect为false时使用）
  "subtitle_region": {
    "x": 0,                          // integer, 字幕区域左上角X坐标, 默认: 0
    "y": 1500,                       // integer, 字幕区域左上角Y坐标
    "width": 1920,                   // integer, 字幕区域宽度
    "height": 100                    // integer, 字幕区域高度
  },
  
  // 去除方法
  "method": "blur",                   // string, 可选值: "blur", "pixelate", "black", "crop"
  "blur_strength": 10,               // integer, 模糊强度, 范围: 1-20, 默认: 10
  "pixelate_size": 15,               // integer, 马赛克大小, 范围: 5-30, 默认: 15
  
  // 异步处理
  "webhook_url": "string"             // 可选, 处理完成后的回调URL
}
```

### 1.2 添加字幕
**接口路径**: `/v1/subtitle/insert-ass`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  "ass": "string",                    // 必填, ASS格式字幕内容
  
  // 字幕定位
  "subtitle_y": 1538,                 // integer, 可选, 字幕Y坐标位置
  "subtitle_height": 77,              // integer, 可选, 字幕高度
  
  // 字幕样式覆盖
  "override_style": {
    "font_name": "Arial",            // string, 字体名称
    "font_size": 48,                 // integer, 字体大小, 范围: 12-200
    "primary_color": "&H00FFFFFF",   // string, ASS颜色格式, 主要颜色
    "outline_color": "&H00000000",   // string, ASS颜色格式, 描边颜色
    "bold": true,                    // boolean, 是否加粗
    "italic": false                  // boolean, 是否斜体
  },
  
  // 输出配置
  "output_format": "mp4",             // string, 输出格式, 默认: "mp4"
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 1.3 自动生成字幕
**接口路径**: `/v1/video/caption`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 视频输入（二选一）
  "video_url": "https://example.com/video.mp4",
  "video": {
    "url": "https://example.com/video.mp4"
  },

  // 可选：手动字幕（不传 captions 则自动从视频音轨识别）
  "captions": "string",

  // 识别语言提示
  "language": "auto",

  // 翻译（可选）
  "translate_to": "ms",
  "bilingual": true,
  "source_lang": "en",

  // 推荐：字幕配置统一写法（兼容 sync 风格）
  "subtitle": {
    "mode": "auto",
    "subtitle_language": "en",
    "subtitle_task": "bilingual",
    "target_lang": "ms",
    "style": "karaoke",
    "font_name": "Arial",
    "primary_color": "#FFFFFF",
    "word_color": "#FFFF00",
    "outline_color": "#000000",
    "outline_width": 3
  },

  // 细粒度样式（可选；会覆盖 subtitle 内同名字段）
  "settings": {
    "style": "karaoke",
    "font_family": "Arial",
    "font_name": "Arial",
    "font_size": 48,

    // 翻译行样式（双语时可用；全片固定，不随句子自动变化）
    "translation_offset_y": 70,
    "translation_font_family": "DejaVu Sans",
    "translation_font_size": 26,
    "translation_font_scale": 0.65,
    "translation_fscx": 92,

    "position": "bottom_center",
    "alignment": "center",
    "x": 0,
    "y": 0,

    "line_color": "#FFFFFF",
    "primary_color": "#FFFFFF",
    "word_color": "#FFFF00",
    "outline_color": "#000000",
    "highlight_color": "#FFFF00",

    "outline_width": 2,
    "shadow_offset": 0,
    "bold": false,
    "italic": false
  },

  "replace": [
    {"find": "old", "replace": "new"}
  ],

  "exclude_time_ranges": [
    {"start": "00:00:01.000", "end": "00:00:03.000"}
  ],

  "output": {
    "filename": "captioned_video.mp4"
  },

  "webhook_url": "string",
  "id": "string"
}
```

**说明（以当前代码实现为准）**：

- 本接口不支持**顶层** `mode`、`engine`、`output_subtitle`、`subtitle_format` 等字段（传入会被拒绝）；字幕模式/引擎请使用 `subtitle.mode` / `subtitle.subtitle_engine`。
- 支持 `output.filename` 用于控制输出文件名（默认 `{job_id}_captioned.mp4`）。
- `captions` 如果是 SRT 文本且 `settings.style!=classic`，会自动降级到 `classic`（避免底层不支持 SRT+特效而失败）。
- 颜色字段支持 `#RRGGBB`，也兼容 ASS 的 `&H..` 格式。
- `translate_to`/`source_lang`/`subtitle.target_lang` 使用标准语言码：日语请使用 `ja`（不要用 `jp`）。
- 翻译行支持独立样式参数：`translation_offset_y`（与主字幕的垂直距离）、`translation_font_family`（翻译字体）、`translation_font_size` 或 `translation_font_scale`（翻译字号）、`translation_fscx`（翻译横向压缩）。

### 1.4 字幕格式转换与动态字幕生成
**接口路径**: `/v1/subtitle/srt-to-ass`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // ==================== 字幕输入（三选一） ====================
  "srt_content": "1\n00:00:00,000 --> 00:00:03,000\n字幕内容\n\n",  // string, 直接传入SRT字幕内容
  // 或者
  "srt_url": "https://example.com/subtitle.srt",  // string, SRT文件URL
  // 或者（无SRT时从视频音轨自动识别）
  "video": {
    "url": "https://example.com/video.mp4",  // string, 视频URL
    // 或者 S3 存储
    "bucket": "my-bucket",  // string, S3桶名
    "key": "path/to/video.mp4"  // string, S3对象key
  },
  
  // ==================== 对齐模式（可选） ====================
  "align_mode": "none",  // string, 字幕对齐: none(不对齐)/whisper(Whisper智能校准)/alass(音频频谱对齐)
  
  // ==================== 渲染引擎选择 ====================
  "render_engine": "pycaps",  // string, "pycaps"(动态字幕) 或 "ass"(传统ASS)
  
  // ==================== PyCaps 动态字幕配置 ====================
  "pycaps": {
    // 字幕风格（一个参数搞定）
    "风格": "炫酷",  // string, 可选值见下方说明
    "style": "hype",  // 英文别名
    
    // 位置控制
    "位置": "底部",   // string, 顶部/中间/底部 (或 top/middle/bottom)
    "position": "bottom",  // 英文别名
    "位置修正": 0,    // integer, Y轴像素偏移，正数向下，负数向上
    "offset": 0,      // 英文别名
    
    // AI 智能功能（需配置 DEEPSEEK_API_KEY）
    "智能分句": false,  // boolean, AI长句拆短句，更适合短视频节奏
    "smart_split": false,  // 英文别名
    "自动表情": false,  // boolean, AI自动添加emoji表情（仅部分模板支持）
    "auto_emoji": false,  // 英文别名
    
    // 颜色覆盖（覆盖模板默认颜色）
    "颜色": "#FF0000",  // string, 十六进制颜色代码
    "color": "#FF0000",  // 英文别名
    
    // 高级：自定义CSS
    "custom_css": "string",  // string, 专业用户可注入自定义CSS样式
    
    // 语言识别（无 SRT 时）
    "语言": "zh",      // string, zh/en/ja/ko/auto
    "language": "zh"   // 英文别名
  },
  
  // ==================== 经典 ASS 样式配置（render_engine="ass" 时使用） ====================
  "style": "classic",  // string, ASS样式模板: classic/karaoke/highlight/tiktok/cinema/youtube等
  "settings": {
    "font_name": "Arial",      // string, 字体名称
    "font_size": 48,           // integer, 字体大小
    "primary_color": "&H00FFFFFF",  // string, 主颜色(ASS格式)
    "outline_color": "&H00000000",  // string, 描边颜色
    "outline_width": 2,        // integer, 描边宽度
    "position": "bottom_center",  // string, 位置: bottom_center/top_center/middle_center
    "margin_v": 50,            // integer, 垂直边距
    "bold": false              // boolean, 是否加粗
    // ...其他 ASS 参数
  },
  
  // ==================== 文本替换（可选） ====================
  "replace": [
    {
      "find": "old_text",      // string, 要查找的文本
      "replace": "new_text"    // string, 替换为的文本
    }
  ],
  
  // ==================== 翻译设置（可选） ====================
  "translate": {
    "enabled": false,          // boolean, 是否启用翻译
    "target_lang": "en",       // string, 目标语言
    "source_lang": "zh",       // string, 源语言
    "bilingual": true          // boolean, 是否生成双语字幕
  },
  
  // ==================== Logo水印（可选） ====================
  "logo": {
    "url": "https://example.com/logo.png",  // string, Logo图片URL
    // 或者 S3
    "bucket": "my-bucket",     // string, S3桶名
    "key": "logo.png",         // string, S3对象key
    
    "position": "top-right",   // string, 位置: top-left/top-right/bottom-left/bottom-right/center
    "scale_height": 80,        // integer, 缩放高度(像素)
    "opacity": 0.8,            // number, 不透明度 (0-1)
    "x_offset": 20,            // integer, X轴偏移(像素)
    "y_offset": 20             // integer, Y轴偏移(像素)
  },
  
  // ==================== 输出配置 ====================
  "output": {
    "burn_video": false,       // boolean, 是否烧录字幕到视频
    "return_srt": true,        // boolean, 是否返回校准后的SRT
    "return_ass": true,        // boolean, 是否返回ASS内容
    "filename": "output.mp4"   // string, 输出文件名
  },
  
  // ==================== 分辨率设置 ====================
  "play_res_x": 1920,          // integer, 播放分辨率宽度
  "play_res_y": 1080,          // integer, 播放分辨率高度
  
  // ==================== 回调与任务ID ====================
  "webhook_url": "https://your-server.com/callback",  // string, 异步回调URL
  "id": "custom_task_id"       // string, 自定义任务ID
}
```

#### PyCaps 风格参数说明

**官方模板（支持表情 🆕）**：
- `炫酷` / `hype`：大字 + Emoji，支持智能表情
- `鲜艳` / `vibrant`：活力配色，支持智能表情

**官方模板（无表情）**：
- `极简` / `minimalist`
- `经典` / `classic`
- `新极简` / `neo_minimal`
- `逐词` / `word-focus`
- `逐行` / `line-focus`

**扩展模板（Walicap）**：
- `治愈` / `hype-healing`：柔和绿色
- `温暖` / `hype-warm`：橙黄渐变
- `恐怖` / `hype-horror`：血红抖动
- `科技` / `hype-tech`：青色数字感
- `电流` / `hype-electric`：紫色闪电
- `浪漫` / `hype-romantic`：粉色梦幻
- `音乐` / `hype-music`：彩虹律动，支持智能表情
- `说唱` / `hype-hiphop`：街头风格
- `派对` / `hype-party`：霓虹闪烁，支持智能表情
- `游戏` / `hype-gaming`：电竞风格，支持智能表情
- `电竞` / `hype-esports`
- `火焰` / `hype-fire`

#### 对齐模式说明
- **none**：不进行对齐，直接使用原始SRT时间轴
- **whisper**：使用Whisper模型重新识别音轨，自动校准时间轴（最准确）
- **alass**：基于音频波形频谱对齐（适合音画不同步的情况）

#### 重要说明
1.  **输入方式**：支持三种方式提供字幕：直接传入SRT内容、提供SRT文件URL、或仅提供视频URL让系统自动识别
2.  **默认渲染引擎**：如不指定 `render_engine`，系统会根据参数自动选择（有 `pycaps` 参数时使用 PyCaps）
3.  **AI 功能依赖**：`智能分句` 和 `自动表情` 需要在服务器配置 `DEEPSEEK_API_KEY` 环境变量
4.  **表情模板**：只有标注"支持智能表情"的模板在开启 `自动表情=true` 时会调用 AI 生成 emoji
5.  **烧录视频**：设置 `output.burn_video=true` 时会将字幕永久烧录到视频中
6.  **参数优先级**：中文参数与英文参数二选一即可，中文参数优先级更高
7.  **颜色格式**：支持 `#RRGGBB` 十六进制格式或 ASS 的 `&H` 格式

---

## 2. 视频处理接口

### 2.1 视频裁剪
**接口路径**: `/v1/video/trim`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  "start_time": "00:00:10",          // string, 必填, 开始时间 (HH:MM:SS 或 秒数)
  "end_time": "00:01:30",            // string, 必填, 结束时间 (HH:MM:SS 或 秒数)
  
  // 编码设置
  "codec": "libx264",                 // string, 视频编码器, 默认: "libx264"
  "quality": "high",                  // string, 质量: "low", "medium", "high", "lossless"
  "crf": 23,                         // integer, 质量因子, 范围: 0-51, 默认: 23
  
  // 输出设置
  "output_format": "mp4",             // string, 输出格式, 默认: "mp4"
  "preserve_metadata": true,          // boolean, 是否保留元数据, 默认: true
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.2 视频拼接
**接口路径**: `/v1/video/concatenate`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_urls": [                     // array, 必填, 视频URL列表
    "string",
    "string"
  ],
  
  // 拼接设置
  "transition": "none",               // string, 转场效果: "none", "fade", "dissolve", "wipe"
  "transition_duration": 0.5,         // number, 转场时长(秒), 范围: 0-5
  "normalize_resolution": true,       // boolean, 是否统一分辨率, 默认: true
  "target_resolution": "1920x1080",   // string, 目标分辨率
  "normalize_fps": true,              // boolean, 是否统一帧率, 默认: true
  "target_fps": 30,                  // integer, 目标帧率
  
  // 音频处理
  "audio_mode": "merge",              // string, 音频模式: "merge", "first", "none"
  "normalize_audio": true,            // boolean, 是否标准化音频, 默认: true
  
  // 输出设置
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.2.1 视频蒙太奇（随机剪辑）
**接口路径**: `/v1/video/montage`  
**请求方法**: `POST`

#### 完整请求体（以当前代码实现为准）
```json
{
  "videos": {
    "bucket": "video-assets",
    "folder_prefix": "shorts/"
  },
  "audio": {
    "bucket": "audio-assets",
    "key": "music.mp3"
  },
  "subtitles": "1\n00:00:00,000 --> 00:00:02,000\n示例字幕\n",
  "auto_subtitle": false,
  "settings": {
    "num_videos": 10,
    "clip_length": 5,
    "shuffle": true,
    "loop_video": true,
    "fps": 30,
    "burn_subtitles": true,
    "subtitle_language": "zh",
    "subtitle_task": "transcribe",
    "subtitle_engine": "whisper"
  },
  "output": {
    "cloud_upload": true,
    "filename": "video_montage.mp4"
  },
  "use_nvidia": true,
  "webhook_url": "string (可选)",
  "id": "string (可选)"
}
```

#### 说明
- **输入来源**：
  - `videos` 支持三选一：
    - S3：`{bucket, folder_prefix}`
    - URL：`{urls: ["http(s)://..."]}`
    - 本地挂载：`{local_folder: "/path/to/folder"}`
  - `audio` 支持三选一：
    - S3：`{bucket, key}`
    - URL：`{url: "http(s)://..."}`
    - 本地挂载：`{local_path: "/path/to/file"}`
- **字幕**：
  - `subtitles`：直接传 SRT 文本（优先）
  - `auto_subtitle=true`：从音频自动生成字幕（使用 `settings.subtitle_language/subtitle_task/subtitle_engine`）
  - `settings.burn_subtitles=true`：会把字幕烧录到视频输出
- **本地挂载安全**：如果设置了环境变量 `LOCAL_MEDIA_ROOT`，则 `local_folder/local_path` 必须位于该目录之内，否则会直接拒绝。
- **编码稳定性**：当 `use_nvidia=true` 且 NVENC 失败时，会自动回退到 `libx264` 继续渲染（避免任务整体失败）。

### 2.3 多视频音频合成
**接口路径**: `/v1/video/combine-audio-with-videos`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  "videos": {
    "bucket": "video-assets",
    "folder_prefix": "wide/"
  },
  "audio": {
    "bucket": "audio-assets",
    "key": "voice.mp3"
  },
  "bg_music": {
    "bucket": "audio-assets",
    "key": "bgm.mp3"
  },
  "subtitles": "1\n00:00:00,000 --> 00:00:05,000\n示例字幕\n",
  "auto_subtitle": false,
  "settings": {
    "max_clip_duration": 10,
    "threads": 4,
    "use_nvidia": true,
    "output_resolution_width": 1920,
    "output_resolution_height": 1080,
    "frame_rate": 30,
    "num_videos": 30,
    "bg_music_volume": 0.4,
    "subtitle_language": "zh",
    "subtitle_task": "transcribe",
    "subtitle_engine": "whisper"
  },
  "output": {
    "cloud_upload": true,
    "filename": "final_video.mp4"
  },
  "webhook_url": "string (可选)",
  "id": "string (可选)"
}
```

#### 说明
- **输入来源**：
  - `videos` 支持三选一：
    - S3：`{bucket, folder_prefix}`
    - URL：`{urls: ["http(s)://..."]}`
    - 本地挂载：`{local_folder: "/path/to/folder"}`
  - `audio` 支持三选一：S3/URL/`local_path`
  - `bg_music`（可选）支持：S3/URL/`local_path`
- **字幕**：
  - `subtitles`：直接传 SRT 文本
  - `auto_subtitle=true` 且未提供 `subtitles`：会从音频自动生成字幕（使用 `settings.subtitle_language/subtitle_task/subtitle_engine`）
- **本地挂载安全**：如果设置了环境变量 `LOCAL_MEDIA_ROOT`，则 `local_folder/local_path` 必须位于该目录之内，否则会直接拒绝。
- **编码稳定性**：当 `settings.use_nvidia=true` 且 NVENC 失败时，会自动回退到 `libx264` 继续渲染（避免任务整体失败）。

### 2.4 视频音频合并
**接口路径**: `/v1/video/merge-audio`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  "audio_url": "string",              // 必填, 音频文件URL
  
  // 音频设置
  "replace_audio": false,             // boolean, true=替换原音频, false=混合音频
  "audio_delay": 0,                   // number, 音频延迟(秒), 可为负值
  "audio_volume": 1.0,                // number, 新音频音量, 范围: 0-2.0
  "original_volume": 0.5,             // number, 原音频音量(replace为false时有效)
  "sync_duration": true,              // boolean, 是否同步时长, 默认: true
  
  // 输出设置
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.5 智能导演 (视频/图片 + 音频合成)
**接口路径**: `/v1/video/audio-with-images`  
**请求方法**: `POST`
**功能**: 智能编排视频/图片素材，使其与音频和字幕完美同步。支持自动变速、镜头切分和背景音效。

#### 完整请求体 (最新版)
```json
{
  "id": "task_custom_id",
  "webhook_url": "https://your-server.com/callback",

  // 1. 素材库配置 (支持图片或视频)
  "images": {
    // 方式 A: S3 存储桶
    "bucket": "assets-bucket",
    "folder_prefix": "story_01/",
    
    // 方式 B: URL 列表
    // "urls": ["http://.../1.mp4", "http://.../2.mp4"],
    
    // 方式 C: 本地目录 (需配置挂载)
    // "local_folder": "/data/assets/story_01"
  },
  
  // 2. 音频配置
  "audio": {
    "url": "http://.../narration.mp3"
  },
  
  // 3. 字幕配置 (智能导演的核心剧本)
  // 支持直接传入 SRT 文件 URL，或纯文本内容
  "subtitles": "http://.../subtitle.srt",
  
  // 4. 背景音乐 (可选)
  "bg_music": {
    "url": "http://.../bgm.mp3"
  },
  
  // 5. 核心设置 (简化版参数)
  "settings": {
    // --- 智能导演开关 ---
    "smart": true,              // boolean, 是否开启智能同步 (推荐 true)
    
    // --- 素材类型 ---
    "type": "video",            // string, "video"(视频) 或 "image"(图片)
    
    // --- 编排规则 ---
    "rule": "order",            // string, "order"(按文件名顺序) 或 "match"(智能内容匹配)
                                // order: 适用于已有分镜顺序的素材
                                // match: 适用于无序素材库，让 AI 自己挑
    
    // --- 其他微调 ---
    "bg_music_volume": 0.2,     // number, 背景音乐音量 (0.0-1.0)
    "image_duration": 5,        // number, 仅当 smart=false 时生效 (默认单图时长)
    "threads": 4,               // integer, 线程数
    "use_nvidia": true          // boolean, 是否使用 GPU 加速
  },
  
  // 6. 输出配置
  "output": {
    "filename": "smart_director_output.mp4",
    "cloud_upload": true
  }
}
```

#### 功能说明
*   **智能变速 (Video Speed Ramping)**:
    *   在 `type="video"` 且 `smart=true` 模式下，系统会自动计算每个镜头需要填充的时长。
    *   如果原视频素材太长，会自动加速播放（上限 5 倍速）。
    *   如果原视频素材太短，会自动慢放或循环。
*   **SRT 驱动**:
    *   系统会根据 SRT 字幕的语意，将碎片的字幕行聚合成完整的“视觉场景 (Scene)”。
    *   这意味着画面不会因为字幕过短而频繁闪烁，而是保持流畅的叙事节奏。
*   **兼容性**:
    *   依然支持旧版参数 (`random_order`, `repeat_to_fill`)，但建议迁移到新版 `settings` 参数以获得最佳效果。


### 2.6 Ken Burns 动态效果
**接口路径**: `/v1/video/effects/ken-burns`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 图片配置
  "images": {
    "urls": [                         // array, 必填, 图片URL列表
      "string"
    ],
    "durations": [5, 6, 4]            // array, 可选, 每张图片的显示时长
  },
  
  // 音频配置
  "audio": {
    "url": "string",                  // string, 可选, 音频URL
    "sync_to_audio": true             // boolean, 是否同步到音频长度
  },
  
  // Ken Burns 效果设置
  "settings": {
    "magnify_factor": 0.5,            // number, 放大因子, 范围: 0.1-2.0, 默认: 0.5
    "fps": 30,                        // integer, 帧率, 默认: 30
    "transition_effect": "fade",      // string, 转场: "fade", "fadeblack", "slideright", "dissolve", "random"
    "transition_duration": 0.5,       // number, 转场时长(秒)
    "zoom_mode": "in",                // string, 缩放模式: "in", "out", "random", "alternate"
    "pan_mode": "random",             // string, 平移模式: "left", "right", "up", "down", "random"
    "pause_at_start": 0.5,            // number, 开始停顿时长(秒)
    "pause_at_end": 0.5               // number, 结束停顿时长(秒)
  },
  
  // 输出设置
  "output_resolution": "1920x1080",   // string, 输出分辨率
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.7 添加Logo水印
**接口路径**: `/v1/video/logo`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  "logo_url": "string",               // 必填, Logo图片URL
  
  // Logo设置
  "position": "top_right",            // string, 位置: "top_left", "top_right", "bottom_left", "bottom_right", "center"
  "size": 100,                        // integer, Logo大小(像素), 范围: 20-500
  "opacity": 1.0,                     // number, 不透明度, 范围: 0.1-1.0
  "margin_x": 20,                     // integer, 水平边距(像素)
  "margin_y": 20,                     // integer, 垂直边距(像素)
  
  // 动画效果
  "fade_in": 0,                       // number, 淡入时长(秒)
  "fade_out": 0,                      // number, 淡出时长(秒)
  "start_time": 0,                    // number, 开始显示时间(秒)
  "end_time": null,                   // number, 结束显示时间(秒), null表示到视频结束
  
  // 输出设置
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.8 添加横幅
**接口路径**: `/v1/video/add-banner`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  "banner_text": "string",            // 必填, 横幅文字内容
  
  // 横幅设置
  "position": "bottom",               // string, 位置: "top", "bottom", "middle"
  "height": 100,                      // integer, 横幅高度(像素)
  "background_color": "#000000",      // string, 背景颜色(十六进制)
  "background_opacity": 0.8,          // number, 背景不透明度, 范围: 0-1.0
  
  // 文字设置
  "font_family": "Arial",             // string, 字体名称
  "font_size": 36,                    // integer, 字体大小
  "text_color": "#FFFFFF",            // string, 文字颜色(十六进制)
  "text_align": "center",             // string, 对齐方式: "left", "center", "right"
  "bold": false,                      // boolean, 是否加粗
  "italic": false,                    // boolean, 是否斜体
  
  // 动画效果
  "scroll": false,                    // boolean, 是否滚动
  "scroll_speed": 50,                 // integer, 滚动速度(scroll为true时有效)
  "fade_in": 0,                       // number, 淡入时长(秒)
  "fade_out": 0,                      // number, 淡出时长(秒)
  
  // 输出设置
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.9 生成缩略图
**接口路径**: `/v1/video/thumbnail`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  
  // 缩略图设置
  "time": "00:00:05",                // string, 截取时间点 (HH:MM:SS 或 秒数)
  "count": 1,                        // integer, 生成缩略图数量, 默认: 1
  "interval": 10,                    // number, 多张缩略图的时间间隔(秒)
  
  // 图片设置
  "width": 320,                      // integer, 缩略图宽度(像素)
  "height": 180,                     // integer, 缩略图高度(像素)
  "quality": 85,                     // integer, JPEG质量, 范围: 1-100
  "format": "jpg",                   // string, 图片格式: "jpg", "png", "webp"
  
  // 输出设置
  "output_as_grid": false,           // boolean, 是否输出为网格图
  "grid_columns": 3,                 // integer, 网格列数(output_as_grid为true时有效)
  "webhook_url": "string"             // 可选, 回调URL
}
```

### 2.10 获取视频信息
**接口路径**: `/v1/video/info`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  
  // 查询选项
  "include_streams": true,            // boolean, 是否包含流信息, 默认: true
  "include_format": true,             // boolean, 是否包含格式信息, 默认: true
  "include_metadata": true            // boolean, 是否包含元数据, 默认: true
}
```

---

**注意事项：**
1. 所有带 `// 注释` 的内容仅用于说明，实际使用时JSON不支持注释，需要删除
2. 参数标记为"必填"的必须提供，其他为可选参数
3. 数值范围和可选值请严格遵守说明中的限制
4. webhook_url 用于异步处理，处理完成后会向该URL发送结果

### 2.11 Lo-Fi 长视频生成 🆕
**接口路径**: `/v1/video/lofi`  
**请求方法**: `POST`  
**功能**: 创建 Lo-Fi 风格长时间视频，支持多张图片均分显示，支持主音乐+背景音效双音轨混音

#### 完整请求体
```json
{
  // 图片配置（images或image二选一，必填）
  "images": {
    "urls": ["图片URL1", "图片URL2"],    // array, 图片URL数组，支持JPG/PNG/GIF/WEBP
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "folder_prefix": "文件夹前缀",       // string, 选填, S3文件夹
    "local_folder": "本地文件夹路径"     // string, 选填, 本地路径
  },
  "image": {
    "url": "图片URL",                     // string, 单图模式URL
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "key": "对象key",                    // string, 选填, S3对象key
    "local_path": "本地文件路径"         // string, 选填, 本地路径
  },
  
  // 音频配置（必填）
  "audio": {
    "urls": ["音频URL1", "音频URL2"],    // array, 必填, 音频文件数组
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "folder_prefix":  "文件夹前缀",       // string, 选填, S3文件夹
    "local_folder": "本地文件夹路径"     // string, 选填, 本地路径
  },
  
  // 背景音效配置（选填）
  "sound_effects": {
    "urls": ["音效URL1", "音效URL2"],    // array, 选填, 背景音效（雨声、咖啡厅氛围等）
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "folder_prefix": "文件夹前缀",       // string, 选填, S3文件夹
    "local_folder": "本地文件夹路径",    // string, 选填, 本地路径
    "volume": 0.3                        // number, 选填, 默认0.3, 音效音量（0-1）
  },
  
  // 设置
  "settings": {
    "video_length": 3600,                // integer, 选填, 默认3600, 视频长度（秒）, 范围60+
    "framerate": 1,                      // number, 选填, 默认1, 帧率, 范围0.1-30
    "loop_audio": true,                  // boolean, 选填, 默认true, 是否循环音频
    "music_volume": 1.0                  // number, 选填, 默认1.0, 主音乐音量（0-1）
  },
  
  // 输出配置
  "output": {
    "cloud_upload": true,                // boolean, 选填, 默认true, 是否上传到云端
    "filename": "lofi_video.mp4"         // string, 选填, 输出文件名
  },
  
  "use_nvidia": true,                    // boolean, 选填, 默认true, 是否使用GPU加速
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **多图模式**: 使用 `images`，多张图片在视频时间轴上均分显示
- **单图模式**: 使用 `image`，单张图片循环显示整个视频时长
- **GIF支持**: GIF文件保持动画效果并循环播放
- **音效混音**: 主音乐和背景音效自动混合，可分别控制音量
- **适用场景**: 学习背景音乐、工作专注音乐、氛围音乐视频（1-24小时）

### 2.12 静态幻灯片视频 🆕
**接口路径**: `/v1/video/static-slideshow`  
**请求方法**: `POST`  
**功能**: 创建静态图片幻灯片视频，支持淡入淡出转场效果

#### 完整请求体
```json
{
  // 图片配置（必填）
  "images": {
    "urls": ["图片URL1", "图片URL2"],    // array, 必填, 图片URL数组
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "folder_prefix": "文件夹前缀",       // string, 选填, S3文件夹
    "local_folder": "本地文件夹路径"     // string, 选填, 本地路径
  },
  
  // 音频配置（必填）
  "audio": {
    "url": "音频URL",                     // string, 必填, 背景音乐
    "bucket": "bucket名称",              // string, 选填, S3存储桶
    "key": "对象key"                     // string, 选填, S3对象key
  },
  
  // 设置
  "settings": {
    "total_video_duration": 300,         // integer, 选填, 默认300, 总视频时长（秒）, 范围10+
    "image_duration": 5.0,               // number, 选填, 默认5.0, 每张图片显示时长（秒）, 范围0.5+
    "transition": "fade"                 // string, 选填, 默认"none", 转场效果: none/fade/slide
  },
  
  // 输出配置
  "output": {
    "cloud_upload": true,                // boolean, 选填, 默认true, 是否上传到云端
    "filename": "slideshow_video.mp4"    // string, 选填, 输出文件名
  },
  
  "use_nvidia": true,                    // boolean, 选填, 默认true, 是否使用GPU加速
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **自动循环**: 图片不足会重复使用，直到达到总时长
- **音频循环**: 音频循环播放匹配视频长度
- **转场类型**: none（无转场）、fade（淡入淡出）、slide（滑动）
- **适用场景**: 商业演示、会议展示、数据可视化

### 2.13 故事视频 🆕
**接口路径**: `/v1/video/story`  
**请求方法**: `POST`  
**功能**: 单张图片配音频和文字字幕生成故事视频

#### 完整请求体
```json
{
  // 图片配置（必填）
  "image": {
    "bucket": "bucket名称",              // string, S3存储桶
    "key": "对象key",                    // string, S3对象key
    "url": "图片URL"                      // string, 图片URL
  },
  
  // 音频配置（必填）
  "audio": {
    "bucket": "bucket名称",              // string, S3存储桶
    "key": "对象key",                    // string, S3对象key
    "url": "音频URL"                      // string, 音频URL
  },
  
  // 故事文本（选填）
  "story": "故事文本内容",               // string, 选填, 要作为字幕显示的文本
  
  // 字幕时长
  "subtitle_duration": 5.0,              // number, 选填, 默认5.0, 每句字幕显示时长（秒）, 范围0.1+
  
  // 输出配置
  "output": {
    "cloud_upload": true,                // boolean, 选填, 默认true, 是否上传到云端
    "filename": "story_video.mp4"        // string, 选填, 默认"story_video.mp4", 输出文件名
  },
  
  "use_nvidia": true,                    // boolean, 选填, 默认true, 是否使用GPU加速
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **单图配音**: 一张图片配合音频和文字字幕
- **字幕自动分段**: story文本会自动分段显示为字幕
- **视频长度**: 自动匹配音频长度
- **适用场景**: 恐怖故事、播客封面、有声书配图

### 2.14 视频循环到音频长度 🆕
**接口路径**: `/v1/video/loop-to-audio`  
**请求方法**: `POST`  
**功能**: 将短视频循环播放，并配合音频，总长度匹配音频时长

#### 完整请求体
```json
{
  // 必填参数
  "video_url": "视频URL",                // string, 必填, 视频文件URL
  "audio_url": "音频URL",                // string, 必填, 音频文件URL
  "audio_duration": 300,                 // number, 必填, 音频时长（秒）, 范围0+
  
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **视频循环**: 短视频会循环播放直到填满音频时长
- **音频替换**: 原视频音频被替换为新音频
- **视频时长**: 最终视频长度等于 `audio_duration`
- **适用场景**: 短视频素材+长音乐、背景循环动画

### 2.15 视频叠加/画中画 🆕
**接口路径**: `/v1/video/overlay`  
**请求方法**: `POST`  
**功能**: 在基础视频上叠加多个视频片段，支持精确时间和位置控制

#### 完整请求体
```json
{
  // 基础视频（必填）
  "base_video": {
    "url": "视频URL",                     // string, 视频URL
    "bucket": "bucket名称",              // string, S3存储桶
    "key": "对象key"                     // string, S3对象key
  },
  
  // 叠加片段数组（必填）
  "overlay_segments": [
    {
      "video": {
        "url": "叠加视频URL",            // string, 叠加视频URL
        "bucket": "bucket名称",          // string, S3存储桶
        "key": "对象key"                 // string, S3对象key
      },
      "start_time": 10,                  // number, 必填, 开始时间（秒）, 范围0+
      "end_time": 20,                    // number, 必填, 结束时间（秒）, 范围0+
      "position": "bottom_right",        // string, 选填, 预设位置: full/top_half/bottom_half/left_half/right_half/custom
      "x_offset": 0,                     // integer, 选填, 默认0, 水平偏移（像素）
      "y_offset": 0,                     // integer, 选填, 默认0, 垂直偏移（像素）
      "width": 640,                      // integer, 选填, 叠加视频宽度（像素）
      "height": 360,                     // integer, 选填, 叠加视频高度（像素）
      "opacity": 1.0                     // number, 选填, 默认1.0, 不透明度（0-1）
    }
  ],
  
  // 输出配置
  "output": {
    "cloud_upload": true,                // boolean, 选填, 默认true, 是否上传到云端
    "filename": "overlay_result.mp4",    // string, 选填, 输出文件名
    "keep_base_audio": true              // boolean, 选填, 默认true, 是否保留底层音频
  },
  
  "use_nvidia": true,                    // boolean, 选填, 默认true, 是否使用GPU加速
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **多层叠加**: 支持多个视频片段在不同时间段叠加
- **位置预设**: 提供常用位置预设（全屏、半屏等）
- **自定义位置**: 使用 `position: "custom"` 配合 x_offset/y_offset/width/height
- **透明度控制**: 可设置叠加视频的透明度
- **音频处理**: 默认保留底层视频音频，叠加视频静音
- **适用场景**: 画中画效果、多机位合成、画面分屏

---

## 3. 字幕处理接口（补充）

### 3.1 字幕分割 🆕
**接口路径**: `/v1/subtitle/split`  
**请求方法**: `POST`  
**功能**: 将字幕按字符数或标点符号分割成多行，优化显示效果

#### 完整请求体
```json
{
  // 必填参数
  "srt_content": "SRT字幕内容",         // string, 必填, SRT格式字幕文本
  
  // 分割设置
  "max_chars": 20,                      // integer, 选填, 默认20, 每行最大字符数, 范围1+
  "method": "punctuation",              // string, 选填, 默认"punctuation", 分割方法: punctuation/greedy/by_word
  
  "webhook_url": "回调URL",              // string, 选填, 任务完成回调地址
  "id": "任务ID"                         // string, 选填, 自定义任务ID
}
```

#### 说明
- **分割方法**:
  - `punctuation`: 按标点符号智能分割
  - `greedy`: 贪婪分割，尽量填满每行
  - `by_word`: 按单词分割
- **保持时间轴**: 分割后保持原有时间轴不变
- **适用场景**: 长字幕优化、多语言字幕换行

---

**注意事项：**
1. 所有带 `// 注释` 的内容仅用于说明，实际使用时JSON不支持注释，需要删除
2. 参数标记为\"必填\"的必须提供，其他为可选参数
3. 数值范围和可选值请严格遵守说明中的限制
4. webhook_url 用于异步处理，处理完成后会向该URL发送结果
5. 🆕 标记表示新增接口（v2.2）

**文档版本**: v2.2  
**最后更新**: 2024-12-25  
**重大更新**: 新增 Lo-Fi、静态幻灯片、故事视频、视频循环、视频叠加、字幕分割等6个接口
