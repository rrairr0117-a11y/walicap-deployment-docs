# ASS 字幕背景框与发光效果实现技术文档

## 📋 目录
1. [问题背景](#问题背景)
2. [核心发现](#核心发现)
3. [正确实现方案](#正确实现方案)
4. [Bug 分析与解决](#bug-分析与解决)
5. [完整代码示例](#完整代码示例)
6. [API 参数说明](#api-参数说明)
7. [常见配色方案](#常见配色方案)

---

## 🎯 问题背景

### **需求**
在 ASS 字幕中实现**半透明背景框**和**动漫风格发光效果**，支持：
- 自定义背景框颜色和透明度
- 自定义发光颜色和强度
- 通过 API 请求体动态配置

### **初始问题**
用户设置了黄色背景框（`box_color: "#FFFF00"`, `box_opacity: 0.6`），但字幕背景始终显示为**黑色**，无法应用自定义颜色。

---

## 🔍 核心发现

### **ASS BorderStyle 参数详解**

| BorderStyle | 名称 | OutlineColour 用途 | BackColour 用途 | Shadow 用途 |
|-------------|------|-------------------|----------------|------------|
| **1** | 描边+阴影 | 文字描边颜色 | 阴影颜色 | 阴影偏移距离 |
| **3** | 不透明背景框 | **背景框颜色** ✅ | **被忽略** ❌ | **被忽略** ❌ |
| **4** | 半透明背景框 | 背景框颜色 | 被忽略 | 被忽略 |

### **关键结论**

> **当 `BorderStyle=3` 时：**
> - `OutlineColour` = 背景框颜色（包含透明度）
> - `Outline` = 背景框大小（像素）
> - `BackColour` = **完全无效**
> - `Shadow` = **必须为 0**

这是 ASS 规范的设计，不是 Bug！

---

## ✅ 正确实现方案

### **1. 背景框模式（BorderStyle=3）**

```python
# 在 create_style_line 函数中
if box_opacity > 0:
    # 计算带透明度的背景颜色
    outline_color_rgb = rgb_to_ass_color(box_color_hex)  # 例如 &H0000FFFF
    alpha = int((1.0 - box_opacity) * 255)  # 0.6 不透明度 → alpha=102
    alpha_hex = format(alpha, '02X')  # '66'
    box_bgr = outline_color_rgb[4:]  # 提取 'FFFF00'
    
    # OutlineColour = &H{alpha}{BBGGRR}
    outline_color = f"&H{alpha_hex}{box_bgr}"  # &H66FFFF00
    
    # BorderStyle=3, Outline=背景框大小
    border_style = '3'
    outline_width = str(box_padding)  # 例如 '8'
    shadow_offset = '0'  # 必须为 0
    
    # BackColour 设为全透明（虽然会被忽略）
    back_color = '&HFF000000'
```

### **2. 发光模式（BorderStyle=1 + blur）**

```python
# 在 create_style_line 函数中
elif glow_color_hex:
    # OutlineColour 用作发光颜色
    outline_color = rgb_to_ass_color(glow_color_hex)  # 例如 &H009314FF (粉红色)
    border_style = '1'  # 描边模式
    outline_width = '2'  # 描边宽度
    shadow_offset = '0'
    back_color = '&HFF000000'

# 在 handle_classic 函数中添加 \blur 标签
if glow_strength > 0:
    background_tag = f"\\blur{glow_strength}"  # 例如 \blur4
```

---

## 🐛 Bug 分析与解决

### **Bug 1: 使用 `\shad` 标签失败**

**错误代码：**
```python
background_tag = f"\\4c{shadow_color}\\4a{alpha_hex}\\shad{box_padding}"
```

**问题：**
- `\4c` 和 `\4a` 只在 Style 的 `Shadow > 0` 时有效
- 当 `Shadow=0` 时，这些标签完全无效

**解决：**
使用 `BorderStyle=3` 在 Style 中设置背景框，而不是在 Dialogue 中使用 override 标签。

---

### **Bug 2: BackColour 设置无效**

**错误代码：**
```python
if box_opacity > 0:
    border_style = '3'
    # 错误：设置 BackColour
    back_color = f"&H{alpha_hex}{box_bgr}"
```

**问题：**
`BorderStyle=3` 时，`BackColour` 参数被 ASS 渲染引擎完全忽略。

**解决：**
将背景颜色设置到 `OutlineColour`，而不是 `BackColour`。

---

### **Bug 3: 颜色格式错误**

**错误代码：**
```python
box_bgr = shadow_color[4:]  # shadow_color = '&H00FFFF00'
# 结果：box_bgr = 'FFFF00' (6位) ✅
# 但如果 shadow_color 格式不对，可能得到错误结果
```

**问题：**
ASS 颜色格式必须严格为 `&HAABBGGRR`（8位），其中：
- `AA` = Alpha（透明度，00=不透明，FF=全透明）
- `BBGGRR` = Blue, Green, Red（BGR 顺序）

**解决：**
```python
def rgb_to_ass_color(hex_color):
    """
    将 RGB 十六进制颜色（如 #FFFF00）转换为 ASS BGR 格式（&H00FFFF00）
    """
    hex_color = hex_color.lstrip('#')
    r = hex_color[0:2]
    g = hex_color[2:4]
    b = hex_color[4:6]
    return f"&H00{b}{g}{r}"  # BGR 顺序，00=不透明
```

---

### **Bug 4: 背景框太小**

**错误代码：**
```python
if box_opacity > 0:
    border_style = '3'
    outline_width = '2'  # 固定为 2px
```

**问题：**
`BorderStyle=3` 时，`Outline` 参数控制背景框大小，而不是描边宽度。固定为 2px 会导致背景框太小。

**解决：**
```python
outline_width = str(box_padding)  # 使用 box_padding 参数（例如 8px）
```

---

### **Bug 5: Docker 容器代码未更新**

**问题：**
修改代码后，Docker 容器仍然运行旧代码，因为：
1. Docker 构建缓存导致 `COPY . /app` 被跳过
2. Gunicorn Worker 进程缓存了旧代码

**解决方案：**

#### **方法 1: 使用 `docker cp` 热更新（推荐用于调试）**
```powershell
# 复制修改后的文件到容器
docker cp "E:\path\to\ass_toolkit.py" ncat:/app/services/ass_toolkit.py

# 强制重启 Gunicorn（杀死所有 worker）
docker exec ncat pkill -9 -f "gunicorn"
```

#### **方法 2: 完全重新构建镜像**
```powershell
# 清理构建缓存
docker builder prune -a -f

# 重新构建
docker-compose build --no-cache ncat
docker-compose up -d ncat
```

---

### **Bug 6: Gunicorn Worker 缓存**

**问题：**
即使使用 `docker cp` 更新了文件，Gunicorn Worker 仍然使用旧代码。

**原因：**
- `pkill -HUP gunicorn` 只是优雅重启，可能不会重新加载所有模块
- `docker restart` 也可能不够

**解决：**
```bash
# 强制杀死所有 Gunicorn 进程
docker exec ncat pkill -9 -f "gunicorn"

# Docker Compose 会自动重启服务
```

---

## 💻 完整代码示例

### **1. `create_style_line` 函数（核心逻辑）**

```python
def create_style_line(style_options, video_resolution):
    """创建 ASS Style 行"""
    
    # 获取参数
    box_color_hex = style_options.get('box_color', '#000000')
    box_opacity = style_options.get('box_opacity', 0.0)
    box_padding = style_options.get('box_padding', 8)
    glow_color_hex = style_options.get('glow_color', None)
    glow_strength = style_options.get('glow_strength', 0)
    user_outline_color = style_options.get('outline_color', '#000000')
    
    # 决定 OutlineColour 的用途
    if box_opacity > 0:
        # 模式 1: 背景框
        outline_color_rgb = rgb_to_ass_color(box_color_hex)
        alpha = int((1.0 - box_opacity) * 255)
        alpha_hex = format(alpha, '02X')
        box_bgr = outline_color_rgb[4:]
        outline_color = f"&H{alpha_hex}{box_bgr}"
        back_color = '&HFF000000'
        border_style = '3'
        outline_width = str(box_padding)
        shadow_offset = '0'
    elif glow_color_hex:
        # 模式 2: 发光效果
        outline_color = rgb_to_ass_color(glow_color_hex)
        back_color = '&HFF000000'
        border_style = '1'
        outline_width = '2'
        shadow_offset = '0'
    else:
        # 模式 3: 普通描边
        outline_color = rgb_to_ass_color(user_outline_color)
        back_color = '&HFF000000'
        border_style = '1'
        outline_width = '2'
        shadow_offset = '0'
    
    # 生成 Style 行
    style_line = (
        f"Style: Default,{font_family},{font_size},{line_color},{secondary_color},"
        f"{outline_color},{back_color},{bold},{italic},{underline},{strikeout},"
        f"{scale_x},{scale_y},{spacing},{angle},{border_style},{outline_width},"
        f"{shadow_offset},{alignment},{margin_l},{margin_r},{margin_v},0"
    )
    
    return style_line
```

### **2. `handle_classic` 函数（添加 blur 效果）**

```python
def handle_classic(transcription_result, style_options, replace_dict, video_resolution):
    """经典样式处理器"""
    
    box_opacity = style_options.get('box_opacity', 0)
    glow_strength = style_options.get('glow_strength', 0)
    
    # 根据模式添加 blur 标签
    if box_opacity > 0:
        # 背景框模式：轻微模糊让边缘更柔和
        background_tag = "\\blur2"
    elif glow_strength > 0:
        # 发光模式：强烈模糊产生发光效果
        background_tag = f"\\blur{glow_strength}"
    else:
        background_tag = ""
    
    # 生成 Dialogue 行
    for segment in transcription_result['segments']:
        position_tag = f"{{\\an{an_code}\\pos({final_x},{final_y})\\q2{background_tag}}}"
        events.append(f"Dialogue: 0,{start_time},{end_time},Default,,0,0,0,,{position_tag}{text}")
    
    return "\n".join(events)
```

### **3. `rgb_to_ass_color` 函数（颜色转换）**

```python
def rgb_to_ass_color(hex_color):
    """
    将 RGB 十六进制颜色转换为 ASS BGR 格式
    
    输入: "#FFFF00" (黄色)
    输出: "&H00FFFF00" (ASS 格式，BGR 顺序)
    """
    hex_color = hex_color.lstrip('#')
    r = hex_color[0:2]
    g = hex_color[2:4]
    b = hex_color[4:6]
    return f"&H00{b}{g}{r}"
```

---

## 📝 API 参数说明

### **所有支持的 ASS 参数**

| 参数 | 类型 | 说明 | 默认值 | 示例 |
|------|------|------|--------|------|
| **文字样式** |
| `primary_color` | string | 文字颜色（RGB 格式） | `#FFFFFF` | `#FFD700` |
| `font_name` | string | 字体名称 | `Arial` | `KingHwa_OldSong` |
| `font_size` | integer | 字体大小（像素） | 自动计算 | `48` |
| `bold` | boolean | 是否加粗 | `false` | `true` |
| `italic` | boolean | 是否斜体 | `false` | `true` |
| `underline` | boolean | 是否下划线 | `false` | `false` |
| `strikeout` | boolean | 是否删除线 | `false` | `false` |
| **描边效果** |
| `outline_color` | string | 描边颜色 | `#000000` | `#8B4513` |
| `outline_width` | integer | 描边宽度（像素） | `2` | `3` |
| **背景框效果** |
| `box_color` | string | 背景框颜色 | `#000000` | `#FFFF00` |
| `box_opacity` | number | 背景框不透明度（0.0-1.0） | `0` | `0.6` |
| `box_padding` | integer | 背景框边距（像素） | `8` | `10` |
| **发光效果** |
| `glow_color` | string | 发光颜色 | `null` | `#FF1493` |
| `glow_strength` | integer | 发光强度（0-10） | `0` | `4` |
| **其他** |
| `target_lang` | string | 目标语言（用于字体匹配） | `en` | `th` |
| `effect` | string | 特效样式 | `classic` | `karaoke` |

### **参数互斥关系**

> **注意：** `box_opacity` 和 `glow_color` 是互斥的！
> - 如果 `box_opacity > 0`，则使用**背景框模式**
> - 如果 `glow_color` 存在，则使用**发光模式**
> - 两者不能同时使用

---

## 🎨 常见配色方案

### **1. 经典黄色背景框**
```json
{
  "subtitle": {
    "primary_color": "#000000",
    "box_color": "#FFFF00",
    "box_opacity": 0.6,
    "box_padding": 8
  }
}
```
**效果：** 黑色文字 + 半透明黄色背景框

---

### **2. 动漫风格粉红发光**
```json
{
  "subtitle": {
    "primary_color": "#FFFFFF",
    "glow_color": "#FF1493",
    "glow_strength": 4,
    "outline_width": 2
  }
}
```
**效果：** 白色文字 + 粉红色发光光晕

---

### **3. 金色文字 + 白色发光**
```json
{
  "subtitle": {
    "primary_color": "#FFD700",
    "glow_color": "#FFFFFF",
    "glow_strength": 4,
    "outline_color": "#000000",
    "outline_width": 2
  }
}
```
**效果：** 金黄色文字 + 白色光晕 + 黑色描边

---

### **4. 霓虹灯效果（蓝色发光）**
```json
{
  "subtitle": {
    "primary_color": "#00FFFF",
    "glow_color": "#0080FF",
    "glow_strength": 5,
    "outline_width": 2
  }
}
```
**效果：** 青色文字 + 蓝色强烈发光

---

### **5. 火焰风格（橙红渐变）**
```json
{
  "subtitle": {
    "primary_color": "#FFA500",
    "glow_color": "#FF4500",
    "glow_strength": 4,
    "outline_color": "#8B0000",
    "outline_width": 2
  }
}
```
**效果：** 橙色文字 + 红色发光 + 深红描边

---

## 🚨 重要注意事项

### **1. 颜色格式**
- API 请求使用 **RGB 格式**：`#RRGGBB`（例如 `#FFFF00`）
- ASS 内部使用 **BGR 格式**：`&HAABBGGRR`（例如 `&H0000FFFF`）
- 转换由 `rgb_to_ass_color` 函数自动完成

### **2. 透明度计算**
```python
# 不透明度 → Alpha 值
alpha = int((1.0 - opacity) * 255)

# 示例
opacity = 0.6  # 60% 不透明
alpha = int((1.0 - 0.6) * 255) = 102
alpha_hex = '66'
```

### **3. BorderStyle 选择**
- **BorderStyle=1**：适合发光效果、普通描边
- **BorderStyle=3**：适合背景框效果
- **不要混用**：同一 Style 只能有一个 BorderStyle

### **4. Docker 开发调试**
开发时推荐使用 `docker cp` + `pkill -9` 快速更新代码：
```powershell
docker cp file.py ncat:/app/path/file.py
docker exec ncat pkill -9 -f "gunicorn"
```

### **5. 性能优化**
- `blur` 值越大，渲染越慢
- 推荐 `glow_strength` 范围：3-5
- 避免在高分辨率视频中使用过大的 `blur` 值

---

## 📚 参考资料

- [Aegisub ASS 标签文档](http://docs.aegisub.org/3.2/ASS_Tags/)
- [ASS 规范（中文）](https://aegi.vmoe.info/docs/3.2/ASS_Tags/)
- [libass 官方仓库](https://github.com/libass/libass)

---

## ✅ 总结

### **核心要点**
1. **BorderStyle=3** 时，`OutlineColour` 是背景框颜色，`BackColour` 无效
2. **发光效果** 通过 `\blur` 标签 + `OutlineColour` 实现
3. **颜色格式** 必须正确转换（RGB → BGR）
4. **Docker 调试** 需要强制重启 Gunicorn Worker

### **已修复的接口**
- ✅ `/v1/video/caption`
- ✅ `/v1/media/generate/ass`
- ✅ `/v1/video/sync-audio-video`

### **测试验证**
```bash
# 查看日志确认参数传递
docker logs ncat --tail 200 | Select-String "box_opacity"
docker logs ncat --tail 200 | Select-String "OutlineColour"
```

---

**文档版本：** v2.0  
**最后更新：** 2024-12-06  
**作者：** Cascade AI Assistant
