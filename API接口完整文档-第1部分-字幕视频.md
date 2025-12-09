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
  // 必填参数
  "video_url": "string",              // 必填, 视频文件URL
  
  // 字幕生成模式
  "mode": "auto",                     // string, "auto"(自动识别), "manual"(使用提供的文本)
  "captions": "string",               // string, mode为manual时必填, 字幕文本内容
  
  // 语言设置
  "language": "zh",                   // string, 语言代码: "zh", "en", "ja", "ko", "auto"
  "translate_to": "en",               // string, 可选, 翻译目标语言
  "bilingual": false,                 // boolean, 是否生成双语字幕, 默认: false
  
  // 识别引擎
  "engine": "whisper",                // string, 可选值: "whisper", "funasr", 默认: "whisper"
  
  // 字幕样式设置
  "settings": {
    "style": "classic",              // string, 可选值: "classic", "karaoke", "typewriter"
    "font_family": "Noto Sans CJK SC", // string, 字体名称
    "font_size": 48,                 // integer, 字体大小, 范围: 12-200
    "position": "bottom_center",     // string, 位置: "bottom_center", "bottom_left", "bottom_right", "top_center"
    "margin_v": 50,                  // integer, 垂直边距, 范围: 0-200, 默认: 50
    
    // 颜色设置 (ASS格式)
    "line_color": "&H00FFFFFF",      // string, 文字颜色
    "word_color": "&H0000FFFF",      // string, 高亮颜色(karaoke模式)
    "outline_color": "&H00000000",   // string, 描边颜色
    "shadow_color": "&H80000000",    // string, 阴影颜色
    
    // 效果设置
    "outline_width": 2,              // integer, 描边宽度, 范围: 0-10
    "shadow_depth": 2,               // integer, 阴影深度, 范围: 0-10
    "bold": false,                   // boolean, 是否加粗
    "italic": false,                 // boolean, 是否斜体
    
    // 时间设置
    "max_duration": 5,               // number, 每句最长显示时间(秒)
    "min_duration": 1                // number, 每句最短显示时间(秒)
  },
  
  // 输出配置
  "output_subtitle": true,            // boolean, 是否输出独立字幕文件
  "subtitle_format": "srt",          // string, 字幕格式: "srt", "ass", "vtt"
  "webhook_url": "string"             // 可选, 回调URL
}
```

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

### 2.3 多视频音频合成
**接口路径**: `/v1/video/combine-audio-with-videos`  
**请求方法**: `POST`

#### 完整请求体
```json
{
  // 视频片段配置
  "video_segments": [
    {
      "video_url": "string",          // string, 必填, 视频URL
      "start_time": 0,                // number, 在最终视频中的开始时间(秒)
      "end_time": 10,                 // number, 在最终视频中的结束时间(秒)
      "trim_start": 0,                // number, 可选, 从源视频的哪个时间点开始(秒)
      "trim_end": null,               // number, 可选, 到源视频的哪个时间点结束(秒)
      "speed": 1.0,                   // number, 播放速度倍率, 范围: 0.5-4.0
      "volume": 1.0                   // number, 音量调整, 范围: 0-2.0
    }
  ],
  
  // 音频配置
  "audio_url": "string",              // string, 可选, 主音频URL
  "bgm_url": "string",                // string, 可选, 背景音乐URL
  "audio_volume": 1.0,                // number, 主音频音量, 范围: 0-2.0
  "bgm_volume": 0.3,                  // number, 背景音乐音量, 范围: 0-2.0
  "audio_fade_in": 0,                 // number, 音频淡入时长(秒)
  "audio_fade_out": 0,                // number, 音频淡出时长(秒)
  
  // 字幕配置
  "auto_subtitle": {
    "enabled": true,                  // boolean, 是否自动生成字幕
    "language": "zh",                 // string, 语言代码
    "style": "classic",              // string, 字幕样式
    "position": "bottom_center"       // string, 字幕位置
  },
  
  // 输出配置
  "output_resolution": "1920x1080",   // string, 输出分辨率
  "output_fps": 30,                   // integer, 输出帧率
  "output_format": "mp4",             // string, 输出格式
  "webhook_url": "string"             // 可选, 回调URL
}
```

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
