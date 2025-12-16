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

### 1.4 字幕格式转换
**接口路径**: `/v1/subtitle/srt-to-ass`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 必填参数
  "srt_content": "string",            // 必填, SRT格式字幕内容
  
  // 样式设置
  "style": "classic",                 // string, 样式模板: "classic", "modern", "minimal"
  
  // 详细样式配置
  "settings": {
    "font_family": "Arial",          // string, 字体名称
    "font_size": 48,                 // integer, 字体大小
    "line_color": "&H00FFFFFF",      // string, 文字颜色(ASS格式)
    "outline_color": "&H00000000",   // string, 描边颜色
    "back_color": "&H80000000",      // string, 背景颜色
    "bold": false,                   // boolean, 是否加粗
    "italic": false,                 // boolean, 是否斜体
    "underline": false,              // boolean, 是否下划线
    "strikeout": false,              // boolean, 是否删除线
    "scale_x": 100,                  // integer, 水平缩放百分比, 范围: 50-200
    "scale_y": 100,                  // integer, 垂直缩放百分比, 范围: 50-200
    "spacing": 0,                    // number, 字符间距
    "angle": 0,                      // number, 旋转角度
    "border_style": 1,               // integer, 边框样式: 1(描边+阴影), 3(背景框)
    "outline": 2,                    // number, 描边宽度
    "shadow": 2,                     // number, 阴影深度
    "alignment": 2,                  // integer, 对齐方式: 1-9(九宫格)
    "margin_l": 10,                  // integer, 左边距
    "margin_r": 10,                  // integer, 右边距
    "margin_v": 10                   // integer, 垂直边距
  }
}
```

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

### 2.5 图片音频生成视频
**接口路径**: `/v1/video/audio-with-images`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 图片配置
  "images": {
    "urls": [                         // array, 必填, 图片URL列表
      "string",
      "string"
    ],
    "duration_per_image": 5,         // number, 每张图片显示时长(秒), 默认: 5
    "random_order": false,            // boolean, 是否随机顺序, 默认: false
    "repeat_to_fill": true           // boolean, 是否重复填充至音频长度, 默认: true
  },
  
  // 音频配置
  "audio": {
    "url": "string",                  // string, 必填, 音频URL
    "trim_start": 0,                  // number, 音频开始时间(秒)
    "trim_end": null,                 // number, 音频结束时间(秒)
    "fade_in": 0,                     // number, 淡入时长(秒)
    "fade_out": 0                     // number, 淡出时长(秒)
  },
  
  // 动画设置
  "settings": {
    "image_duration": 10,             // number, 图片显示时长(覆盖单独设置)
    "num_images": 30,                 // integer, 使用图片数量
    "transition": "fade",             // string, 转场效果: "none", "fade", "slide", "zoom"
    "transition_duration": 0.5,       // number, 转场时长(秒)
    "ken_burns": false,               // boolean, 是否启用Ken Burns效果
    "zoom_factor": 1.2,               // number, 缩放因子(ken_burns为true时有效)
    "pan_direction": "random"         // string, 平移方向: "left", "right", "up", "down", "random"
  },
  
  // 输出设置
  "output_resolution": "1920x1080",   // string, 输出分辨率
  "output_fps": 30,                   // integer, 输出帧率
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

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
