# ASS 字幕背景框实现技术文档

## 📋 目录

- [问题背景](#问题背景)
- [需求说明](#需求说明)
- [技术方案](#技术方案)
- [核心发现](#核心发现)
- [实现细节](#实现细节)
- [Bug 分析与解决](#bug-分析与解决)
- [完整代码示例](#完整代码示例)
- [测试验证](#测试验证)
- [注意事项](#注意事项)

---

## 问题背景

### 初始需求
为 ASS 字幕添加可自定义颜色和透明度的半透明背景框，支持以下参数：
- `box_color`: 背景颜色（十六进制，如 `#FFFF00` 黄色）
- `box_opacity`: 背景不透明度（0.0-1.0，如 `0.6` 表示 60% 不透明）
- `box_padding`: 背景框边距（像素，如 `8`）

### 初始尝试
最初使用 ASS 的 `\shad` 标签（阴影）配合 `\4c`（阴影颜色）和 `\4a`（阴影透明度）来模拟背景框：

```ass
{\shad8\4c&H0000FFFF\4a&H66&}文字内容
```

**结果：失败** ❌
- 阴影始终显示为黑色
- `\4c` 标签无法改变阴影颜色
- `\shad` 标签的阴影颜色是固定的，无法通过 override 标签修改

---

## 需求说明

### 功能要求
1. 支持自定义背景颜色（RGB 十六进制格式）
2. 支持自定义透明度（0.0-1.0）
3. 支持自定义背景框大小（像素）
4. 背景框应该紧贴文字，不影响文字描边
5. 支持双语字幕（中文 + 目标语言）

### API 参数
```json
{
  "subtitle": {
    "box_color": "#FFFF00",      // 背景颜色（黄色）
    "box_opacity": 0.6,          // 60% 不透明
    "box_padding": 8             // 8 像素边距
  }
}
```

---

## 技术方案

### ASS BorderStyle 参数详解

ASS 字幕的 `BorderStyle` 参数控制文字边框和背景的渲染方式：

| BorderStyle | 名称 | OutlineColour 用途 | BackColour 用途 | Shadow 用途 |
|-------------|------|-------------------|----------------|------------|
| **1** | 描边 + 阴影（默认） | 文字描边颜色 | 不显示 | 阴影（黑色固定） |
| **3** | 不透明背景框 ✅ | **背景框颜色** | 不显示 | 不使用 |
| **4** | 半透明背景框（已弃用） | 背景框颜色 | 不显示 | 不使用 |

### ⚠️ 核心发现

**BorderStyle=3 的关键特性：**

1. **`OutlineColour`（第3个颜色参数）控制背景框颜色，而不是 `BackColour`！**
2. **`Outline` 参数控制背景框的大小（像素）**
3. **`BackColour` 在 BorderStyle=3 时完全被忽略**
4. **`Shadow` 参数应设为 `0`，避免额外阴影**

---

## 核心发现

### ASS Style 行格式

```
Style: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
```

### 颜色参数位置

| 位置 | 参数名 | BorderStyle=1 用途 | BorderStyle=3 用途 |
|------|--------|-------------------|-------------------|
| 第1个 | PrimaryColour | 文字颜色 | 文字颜色 |
| 第2个 | SecondaryColour | 卡拉OK高亮颜色 | 卡拉OK高亮颜色 |
| 第3个 | **OutlineColour** | 文字描边颜色 | **背景框颜色** ✅ |
| 第4个 | BackColour | 不显示 | 不显示 |

### 正确的参数配置

**启用背景框时（box_opacity > 0）：**

```python
BorderStyle = 3              # 启用背景框模式
OutlineColour = &H{AA}{BBGGRR}  # 背景框颜色（带透明度）
Outline = box_padding        # 背景框大小（像素）
Shadow = 0                   # 禁用阴影
BackColour = &HFF000000      # 全透明黑色（不使用）
```

**不启用背景框时（box_opacity = 0）：**

```python
BorderStyle = 1              # 默认描边模式
OutlineColour = &H00{BBGGRR}   # 文字描边颜色
Outline = 2                  # 描边宽度
Shadow = 0                   # 禁用阴影
BackColour = &HFF000000      # 全透明黑色
```

---

## 实现细节

### 1. 颜色格式转换

#### RGB 十六进制 → ASS BGR 格式

```python
def rgb_to_ass_color(hex_color):
    """
    将 RGB 十六进制颜色（如 #FFFF00）转换为 ASS BGR 格式（如 &H0000FFFF）
    
    ASS 颜色格式：&H00BBGGRR
    - 00: Alpha（固定为00，表示不透明）
    - BB: Blue（蓝色分量）
    - GG: Green（绿色分量）
    - RR: Red（红色分量）
    """
    hex_color = hex_color.lstrip('#')
    r = hex_color[0:2]
    g = hex_color[2:4]
    b = hex_color[4:6]
    return f"&H00{b.upper()}{g.upper()}{r.upper()}"

# 示例
rgb_to_ass_color("#FFFF00")  # 返回 "&H0000FFFF" (黄色)
rgb_to_ass_color("#FF0000")  # 返回 "&H000000FF" (红色)
rgb_to_ass_color("#00FF00")  # 返回 "&H0000FF00" (绿色)
```

#### 添加透明度

```python
def add_alpha_to_ass_color(ass_color, opacity):
    """
    为 ASS 颜色添加透明度
    
    参数:
        ass_color: ASS 颜色格式 &H00BBGGRR
        opacity: 不透明度 0.0-1.0（0.6 表示 60% 不透明，40% 透明）
    
    返回:
        &HAABBGGRR 格式的颜色
    """
    # 计算 Alpha 值：opacity=0.6 表示 60% 不透明，即 40% 透明
    # ASS Alpha: 00=不透明, FF=全透明
    alpha = int((1.0 - opacity) * 255)
    alpha_hex = format(alpha, '02X')
    
    # 提取 BBGGRR 部分（跳过 &H00）
    bgr = ass_color[4:]  # 从第4个字符开始，提取6位 BBGGRR
    
    return f"&H{alpha_hex}{bgr}"

# 示例
add_alpha_to_ass_color("&H0000FFFF", 0.6)  # 返回 "&H6600FFFF" (60%不透明的黄色)
add_alpha_to_ass_color("&H0000FFFF", 1.0)  # 返回 "&H0000FFFF" (完全不透明的黄色)
add_alpha_to_ass_color("&H0000FFFF", 0.0)  # 返回 "&HFF00FFFF" (完全透明的黄色)
```

### 2. Style 行生成逻辑

```python
def create_style_line(style_options, video_resolution):
    """
    生成 ASS Style 行
    """
    # 获取背景框参数
    box_color_hex = style_options.get('box_color', '#000000')
    box_opacity = style_options.get('box_opacity', 0.0)
    box_padding = style_options.get('box_padding', 8)
    
    # 获取用户指定的描边颜色
    user_outline_color = style_options.get('outline_color', '#000000')
    
    # 根据是否启用背景框，决定 OutlineColour 的用途
    if box_opacity > 0:
        # 启用背景框：OutlineColour 用作背景框颜色
        outline_color_rgb = rgb_to_ass_color(box_color_hex)
        outline_color = add_alpha_to_ass_color(outline_color_rgb, box_opacity)
        
        border_style = '3'  # 背景框模式
        outline_width = str(box_padding)  # 背景框大小
        shadow_offset = '0'  # 禁用阴影
    else:
        # 不启用背景框：OutlineColour 用作文字描边颜色
        outline_color = rgb_to_ass_color(user_outline_color)
        
        border_style = '1'  # 默认描边模式
        outline_width = style_options.get('outline_width', '2')
        shadow_offset = '0'
    
    # BackColour 始终设为全透明（BorderStyle=3 时不使用）
    back_color = '&HFF000000'
    
    # 其他颜色参数
    line_color = rgb_to_ass_color(style_options.get('line_color', '#FFFFFF'))
    secondary_color = rgb_to_ass_color(style_options.get('word_color', '#FFFF00'))
    
    # 生成 Style 行
    style_line = (
        f"Style: Default,{font_family},{font_size},{line_color},{secondary_color},"
        f"{outline_color},{back_color},{bold},{italic},{underline},{strikeout},"
        f"{scale_x},{scale_y},{spacing},{angle},{border_style},{outline_width},"
        f"{shadow_offset},{alignment},{margin_l},{margin_r},{margin_v},0"
    )
    
    return style_line
```

### 3. 完整示例

#### 输入参数
```python
style_options = {
    'box_color': '#FFFF00',      # 黄色
    'box_opacity': 0.6,          # 60% 不透明
    'box_padding': 8,            # 8 像素边距
    'line_color': '#FFFFFF',     # 白色文字
    'outline_color': '#000000',  # 黑色描边（不使用）
    'font_family': 'KingHwa_OldSong',
    'font_size': 35
}
```

#### 生成的 Style 行
```
Style: Default,KingHwa_OldSong,35,&H00FFFFFF,&H0000FFFF,&H6600FFFF,&HFF000000,1,0,0,0,100,100,-1,0,3,8,0,5,20,20,20,0
```

#### 参数解析
| 参数位置 | 参数名 | 值 | 说明 |
|---------|--------|---|------|
| 1 | Name | Default | 样式名称 |
| 2 | Fontname | KingHwa_OldSong | 字体名称 |
| 3 | Fontsize | 35 | 字体大小 |
| 4 | PrimaryColour | &H00FFFFFF | 白色文字 |
| 5 | SecondaryColour | &H0000FFFF | 青色（卡拉OK） |
| 6 | **OutlineColour** | **&H6600FFFF** | **黄色背景（60%不透明）** ✅ |
| 7 | BackColour | &HFF000000 | 全透明黑色（不使用） |
| 8-14 | Bold...Angle | 1,0,0,0,100,100,-1,0 | 字体样式 |
| 15 | **BorderStyle** | **3** | **背景框模式** ✅ |
| 16 | **Outline** | **8** | **背景框大小 8px** ✅ |
| 17 | Shadow | 0 | 禁用阴影 |
| 18-22 | Alignment...Encoding | 5,20,20,20,0 | 对齐和边距 |

---

## Bug 分析与解决

### Bug 1: 使用 `\shad` 标签实现背景框失败

#### 问题描述
初始方案使用 ASS override 标签 `\shad` 配合 `\4c` 和 `\4a` 来创建背景框：

```ass
{\shad8\4c&H0000FFFF\4a&H66&}文字内容
```

**现象：** 背景始终显示为黑色，无法改变颜色。

#### 原因分析
1. **`\shad` 标签的阴影颜色是固定的**，默认为黑色
2. **`\4c` 标签用于控制 Shadow 颜色**，但只在 Style 中的 `Shadow` 参数 > 0 时才生效
3. **当 Style 中 `Shadow=0` 时，`\4c` 标签完全无效**
4. **ASS 规范中，阴影颜色无法通过 override 标签动态修改**

#### 解决方案
放弃使用 `\shad` 标签，改用 **BorderStyle=3** 的背景框模式。

---

### Bug 2: BackColour 设置无效

#### 问题描述
将 `BackColour` 设置为黄色半透明（`&H6600FFFF`），但背景仍然是黑色。

```python
# 错误的实现
style_back_color = f"&H{alpha_hex}{box_bgr}"  # &H6600FFFF
style_line = f"Style: ...,{outline_color},{style_back_color},..."
```

**现象：** 背景框显示为黑色，而不是黄色。

#### 原因分析
**BorderStyle=3 时，`BackColour` 参数完全被忽略！**

ASS 规范明确指出：
- BorderStyle=1: BackColour 不显示
- **BorderStyle=3: BackColour 不显示，背景框颜色由 `OutlineColour` 控制**
- BorderStyle=4: BackColour 不显示

#### 解决方案
**将背景框颜色设置到 `OutlineColour` 参数，而不是 `BackColour`！**

```python
# 正确的实现
if box_opacity > 0:
    # OutlineColour 用作背景框颜色
    outline_color = f"&H{alpha_hex}{box_bgr}"  # &H6600FFFF
    back_color = '&HFF000000'  # BackColour 设为全透明（不使用）
```

---

### Bug 3: 背景框颜色格式错误

#### 问题描述
生成的 `OutlineColour` 格式为 `&H660000FFFF`（10位），导致颜色解析错误。

```python
# 错误的实现
box_bgr = shadow_color[2:]  # 从 &H0000FFFF 提取，得到 "0000FFFF"（8位）
style_back_color = f"&H{alpha_hex}{box_bgr}"  # &H660000FFFF（10位）❌
```

**现象：** 背景框显示为黑色或其他错误颜色。

#### 原因分析
ASS 颜色格式必须是 **8位十六进制**：`&HAABBGGRR`
- `AA`: Alpha（2位）
- `BBGGRR`: 颜色（6位）

错误的实现从 `&H0000FFFF` 提取了 `0000FFFF`（8位），再加上 `66`，变成了 10 位。

#### 解决方案
**正确提取 BBGGRR 部分（6位）：**

```python
# 正确的实现
outline_color_rgb = rgb_to_ass_color(box_color_hex)  # &H0000FFFF
box_bgr = outline_color_rgb[4:]  # 跳过 &H00，提取 "00FFFF"（6位）✅
outline_color = f"&H{alpha_hex}{box_bgr}"  # &H6600FFFF（8位）✅
```

---

### Bug 4: 背景框太小，几乎看不见

#### 问题描述
设置了 `BorderStyle=3` 和正确的颜色，但背景框非常小，几乎看不见。

```python
# 错误的实现
border_style = '3'
outline_width = style_options.get('outline_width', '2')  # 只有 2 像素
```

**现象：** 背景框存在，但只有 2 像素宽，几乎看不见。

#### 原因分析
**BorderStyle=3 时，`Outline` 参数控制背景框的大小（像素）。**

默认的 `outline_width=2` 只适用于文字描边，对于背景框来说太小了。

#### 解决方案
**当启用背景框时，将 `Outline` 设置为 `box_padding` 的值（如 8 像素）：**

```python
# 正确的实现
if box_opacity > 0:
    border_style = '3'
    outline_width = str(box_padding)  # 使用 box_padding（如 8）✅
else:
    border_style = '1'
    outline_width = style_options.get('outline_width', '2')
```

---

### Bug 5: Docker 容器代码未更新

#### 问题描述
修改了本地代码，重新构建了 Docker 镜像，但容器内运行的仍然是旧代码。

**现象：** 日志中没有新增的调试信息，背景颜色没有变化。

#### 原因分析
1. **Docker 构建缓存：** `COPY` 步骤被标记为 `CACHED`，没有复制新文件
2. **Gunicorn 预加载：** Gunicorn 使用了预加载模式，代码在启动时加载到内存，重启容器也不会重新读取文件
3. **Worker 进程缓存：** 即使复制了新文件，worker 进程仍在使用旧的 Python 模块

#### 解决方案

**方案 1：强制无缓存构建（推荐用于正式发布）**
```bash
docker build --no-cache -t walicap:v2.1 .
```

**方案 2：热更新文件到容器（推荐用于开发调试）**
```bash
# 1. 复制文件到容器
docker cp "本地文件路径" 容器名:/app/目标路径

# 2. 强制杀死所有 Gunicorn worker 进程
docker exec 容器名 pkill -9 -f "gunicorn"

# 3. 等待 5-10 秒让 Gunicorn 自动重启 worker
```

**方案 3：完全重启容器**
```bash
docker restart 容器名
```

#### 验证方法
检查容器内的文件是否更新：
```bash
docker exec 容器名 grep -n "关键代码" /app/services/ass_toolkit.py
```

---

### Bug 6: Gunicorn Worker 未重新加载代码

#### 问题描述
使用 `docker cp` 复制了新文件到容器，但运行的仍然是旧代码。

**现象：** 日志中没有新的调试信息。

#### 原因分析
Gunicorn 的 worker 进程在启动时加载 Python 模块，并缓存在内存中。即使文件被替换，worker 也不会自动重新加载。

常见的重载信号（如 `SIGHUP`）只会重启 master 进程，不一定会重新加载所有 worker。

#### 解决方案
**强制杀死所有 worker 进程，让 Gunicorn 重新 fork：**

```bash
# 杀死所有 gunicorn 进程（包括 master 和 worker）
docker exec ncat pkill -9 -f "gunicorn"

# 容器的启动脚本会自动重启 Gunicorn
```

**验证 worker 是否重启：**
```bash
docker exec ncat ps aux
# 检查 gunicorn worker 的 START 时间是否是最新的
```

---

## 完整代码示例

### 完整的 `create_style_line` 函数

```python
def create_style_line(style_options, video_resolution):
    """
    生成 ASS Style 行，支持自定义背景框
    
    参数:
        style_options: 样式配置字典
            - box_color: 背景颜色（如 "#FFFF00"）
            - box_opacity: 背景不透明度（0.0-1.0）
            - box_padding: 背景框边距（像素）
            - line_color: 文字颜色
            - outline_color: 描边颜色（仅在不启用背景框时使用）
            - font_family: 字体名称
            - font_size: 字体大小
        video_resolution: 视频分辨率 (width, height)
    
    返回:
        ASS Style 行字符串
    """
    # 获取字体参数
    target_lang = style_options.get('target_lang', 'en')
    available_fonts = get_available_fonts()
    font_family = style_options.get('font_family', 'Arial')
    
    # 字体回退逻辑
    if font_family not in available_fonts:
        logger.warning(f"Font '{font_family}' not found, trying fallback...")
        font_family = get_fallback_font(target_lang, available_fonts)
        if font_family not in available_fonts:
            logger.error(f"No suitable font found!")
            return {'error': f"Font '{font_family}' not available.", 'available_fonts': available_fonts}
    
    # 文字颜色
    line_color = rgb_to_ass_color(style_options.get('line_color', '#FFFFFF'))
    word_color = style_options.get('word_color', '#FFFF00')
    secondary_color = rgb_to_ass_color(word_color)
    
    # 背景框参数
    box_color_hex = style_options.get('box_color', '#000000')
    box_opacity = style_options.get('box_opacity', 0.0)
    box_padding = style_options.get('box_padding', 8)
    
    logger.info(f"[create_style_line] box_color_hex={box_color_hex}, box_opacity={box_opacity}, box_padding={box_padding}")
    
    # 用户指定的描边颜色
    user_outline_color = style_options.get('outline_color', '#000000')
    
    # 根据是否启用背景框，决定 OutlineColour 的用途
    if box_opacity > 0:
        # 启用背景框：OutlineColour 用作背景框颜色
        outline_color_rgb = rgb_to_ass_color(box_color_hex)
        
        # 计算带透明度的背景颜色
        alpha = int((1.0 - box_opacity) * 255)
        alpha_hex = format(alpha, '02X')
        box_bgr = outline_color_rgb[4:]  # 提取 BBGGRR（6位）
        outline_color = f"&H{alpha_hex}{box_bgr}"
        
        back_color = '&HFF000000'  # BackColour 全透明（不使用）
        
        border_style = '3'  # 背景框模式
        outline_width = str(box_padding)  # 背景框大小
        shadow_offset = '0'  # 禁用阴影
    else:
        # 不启用背景框：OutlineColour 用作文字描边颜色
        outline_color = rgb_to_ass_color(user_outline_color)
        back_color = '&HFF000000'
        
        border_style = style_options.get('border_style', '1')
        outline_width = style_options.get('outline_width', '2')
        shadow_offset = '0'
    
    # 字体样式参数
    default_font_size = int(video_resolution[1] * 0.028)
    font_size = style_options.get('font_size', default_font_size)
    bold = '1' if style_options.get('bold', False) else '0'
    italic = '1' if style_options.get('italic', False) else '0'
    underline = '1' if style_options.get('underline', False) else '0'
    strikeout = '1' if style_options.get('strikeout', False) else '0'
    scale_x = style_options.get('scale_x', '100')
    scale_y = style_options.get('scale_y', '100')
    spacing = style_options.get('spacing', '-1')
    angle = style_options.get('angle', '0')
    
    # 边距参数
    margin_l = style_options.get('margin_l', '20')
    margin_r = style_options.get('margin_r', '20')
    margin_v = style_options.get('margin_v', '20')
    
    # 对齐方式
    alignment = 5
    
    logger.info(f"[OutlineColour] box_opacity={box_opacity}, outline_color={outline_color}, back_color={back_color}")
    
    # 生成 Style 行
    style_line = (
        f"Style: Default,{font_family},{font_size},{line_color},{secondary_color},"
        f"{outline_color},{back_color},{bold},{italic},{underline},{strikeout},"
        f"{scale_x},{scale_y},{spacing},{angle},{border_style},{outline_width},"
        f"{shadow_offset},{alignment},{margin_l},{margin_r},{margin_v},0"
    )
    
    logger.info(f"[ASS Style Debug] Created style line: {style_line}")
    logger.info(f"[ASS Style Debug] PrimaryColour (line_color): {line_color}")
    logger.info(f"[ASS Style Debug] SecondaryColour (word_color): {secondary_color}")
    logger.info(f"[ASS Style Debug] OutlineColour: {outline_color}")
    
    return style_line
```

### 辅助函数

```python
def rgb_to_ass_color(hex_color):
    """
    将 RGB 十六进制颜色转换为 ASS BGR 格式
    
    参数:
        hex_color: RGB 十六进制颜色（如 "#FFFF00" 或 "FFFF00"）
    
    返回:
        ASS 颜色格式 &H00BBGGRR
    """
    hex_color = hex_color.lstrip('#')
    r = hex_color[0:2]
    g = hex_color[2:4]
    b = hex_color[4:6]
    return f"&H00{b.upper()}{g.upper()}{r.upper()}"


def get_available_fonts():
    """
    获取系统可用字体列表
    
    返回:
        字体名称集合
    """
    import matplotlib.font_manager as fm
    fonts = set([f.name for f in fm.fontManager.ttflist])
    return fonts


def get_fallback_font(target_lang, available_fonts, is_chinese=False):
    """
    根据目标语言选择合适的后备字体
    
    参数:
        target_lang: 目标语言代码（如 'th', 'ar', 'ja'）
        available_fonts: 可用字体集合
        is_chinese: 是否为中文
    
    返回:
        字体名称
    """
    if is_chinese:
        chinese_fonts = [
            'KingHwa_OldSong',
            'Noto Sans TC',
            'Noto Sans CJK SC',
            'WenQuanYi Zen Hei',
            'Arial',
        ]
        for font in chinese_fonts:
            if font in available_fonts:
                return font
    
    # 语言特定字体映射
    lang_specific_fonts = {
        'th': ['Noto Sans Thai', 'NotoSansThai', 'Sarabun', 'Tahoma'],
        'ar': ['Noto Sans Arabic', 'NotoSansArabic', 'Arial', 'Tahoma'],
        'ja': ['Noto Sans CJK SC', 'NotoSansCJKsc', 'MS Gothic'],
        'ko': ['Noto Sans CJK SC', 'NotoSansCJKsc', 'Malgun Gothic'],
        # ... 更多语言
    }
    
    if target_lang in lang_specific_fonts:
        for font in lang_specific_fonts[target_lang]:
            if font in available_fonts:
                return font
    
    # 默认字体
    return 'Arial'
```

---

## 测试验证

### 测试用例 1: 黄色半透明背景

**输入：**
```json
{
  "box_color": "#FFFF00",
  "box_opacity": 0.6,
  "box_padding": 8
}
```

**预期输出：**
```
Style: ...,&H6600FFFF,&HFF000000,...,3,8,0,...
```

**验证点：**
- OutlineColour = `&H6600FFFF` (66透明度 + 黄色)
- BorderStyle = `3`
- Outline = `8`
- Shadow = `0`

**实际效果：**
- ✅ 字幕显示黄色半透明背景
- ✅ 背景框大小适中（8像素边距）
- ✅ 文字清晰可见

---

### 测试用例 2: 红色完全不透明背景

**输入：**
```json
{
  "box_color": "#FF0000",
  "box_opacity": 1.0,
  "box_padding": 10
}
```

**预期输出：**
```
Style: ...,&H000000FF,&HFF000000,...,3,10,0,...
```

**验证点：**
- OutlineColour = `&H000000FF` (00透明度 + 红色 = 完全不透明)
- Outline = `10`

**实际效果：**
- ✅ 字幕显示红色不透明背景
- ✅ 背景框更大（10像素边距）

---

### 测试用例 3: 不启用背景框

**输入：**
```json
{
  "box_opacity": 0,
  "outline_color": "#000000",
  "outline_width": 2
}
```

**预期输出：**
```
Style: ...,&H00000000,&HFF000000,...,1,2,0,...
```

**验证点：**
- OutlineColour = `&H00000000` (黑色描边)
- BorderStyle = `1` (默认描边模式)
- Outline = `2` (描边宽度)

**实际效果：**
- ✅ 字幕显示黑色描边，无背景框
- ✅ 描边宽度为 2 像素

---

### 日志验证

**启用背景框时的日志：**
```
INFO:services.ass_toolkit:[create_style_line] box_color_hex=#FFFF00, box_opacity=0.6, box_padding=8
INFO:services.ass_toolkit:[OutlineColour] box_opacity=0.6, outline_color=&H6600FFFF, back_color=&HFF000000
INFO:services.ass_toolkit:[ASS Style Debug] Created style line: Style: Default,KingHwa_OldSong,35,&H00FFFFFF,&H0000FFFF,&H6600FFFF,&HFF000000,1,0,0,0,100,100,-1,0,3,8,0,5,20,20,20,0
INFO:services.ass_toolkit:[ASS Style Debug] PrimaryColour (line_color): &H00FFFFFF
INFO:services.ass_toolkit:[ASS Style Debug] SecondaryColour (word_color): &H0000FFFF
INFO:services.ass_toolkit:[ASS Style Debug] OutlineColour: &H6600FFFF
```

**关键验证点：**
- ✅ `box_opacity=0.6` 正确传递
- ✅ `outline_color=&H6600FFFF` 正确计算（66透明度 + 黄色）
- ✅ BorderStyle=3, Outline=8, Shadow=0

---

## 注意事项

### 1. 颜色格式

- **输入格式：** RGB 十六进制（`#RRGGBB`），如 `#FFFF00`
- **ASS 格式：** BGR 十六进制（`&HAABBGGRR`），如 `&H6600FFFF`
- **转换顺序：** RGB → BGR，然后添加透明度

### 2. 透明度计算

- **输入：** `box_opacity`（0.0-1.0），表示**不透明度**
- **ASS Alpha：** 00=不透明，FF=全透明
- **计算公式：** `alpha = int((1.0 - box_opacity) * 255)`
- **示例：**
  - `box_opacity=0.6` → `alpha=102` → `alpha_hex=66`
  - `box_opacity=1.0` → `alpha=0` → `alpha_hex=00`
  - `box_opacity=0.0` → `alpha=255` → `alpha_hex=FF`

### 3. BorderStyle 选择

| 场景 | BorderStyle | OutlineColour 用途 | Outline 用途 |
|------|-------------|-------------------|-------------|
| 需要背景框 | 3 | 背景框颜色 | 背景框大小 |
| 需要文字描边 | 1 | 描边颜色 | 描边宽度 |
| 两者都需要 | ❌ 不支持 | - | - |

**⚠️ 重要：** BorderStyle=3 时，**无法同时显示文字描边和背景框**。如果需要两者，可以考虑使用双层字幕或其他方案。

### 4. 背景框大小

- **推荐值：** 6-10 像素
- **太小（<4）：** 背景框几乎看不见
- **太大（>15）：** 背景框过大，影响美观
- **自适应：** 可以根据字体大小动态计算，如 `box_padding = font_size * 0.25`

### 5. Docker 开发调试

**推荐工作流：**

1. **开发阶段：** 使用 `docker cp` + `pkill` 热更新代码
   ```bash
   docker cp 本地文件 容器:/app/路径
   docker exec 容器 pkill -9 -f "gunicorn"
   ```

2. **测试阶段：** 完全重启容器
   ```bash
   docker restart 容器名
   ```

3. **发布阶段：** 无缓存重新构建镜像
   ```bash
   docker build --no-cache -t 镜像名:版本 .
   ```

### 6. 兼容性

- **ASS 版本：** 本方案基于 ASS v4.00+ 规范
- **渲染引擎：** 已测试 libass（FFmpeg）
- **其他引擎：** 可能需要调整参数

### 7. 性能考虑

- **背景框渲染：** BorderStyle=3 的性能优于使用 `\shad` 标签
- **大量字幕：** 建议使用 Style 定义，而不是每个 Dialogue 都使用 override 标签

---

## 总结

### 核心要点

1. **BorderStyle=3 是实现背景框的正确方法**
2. **OutlineColour 控制背景框颜色，而不是 BackColour**
3. **Outline 控制背景框大小**
4. **颜色格式必须是 8 位十六进制：`&HAABBGGRR`**
5. **透明度计算：`alpha = int((1.0 - opacity) * 255)`**

### 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| 背景始终是黑色 | 使用了 `\shad` 标签 | 改用 BorderStyle=3 |
| BackColour 设置无效 | BorderStyle=3 不使用 BackColour | 设置 OutlineColour |
| 颜色格式错误 | 提取了错误的字符数 | 从第4个字符开始提取6位 |
| 背景框太小 | Outline 值太小 | 设置为 box_padding 值 |
| 代码未更新 | Docker 缓存或 Gunicorn 缓存 | 强制重启 worker 进程 |

### 参考资料

- [ASS 字幕格式规范](http://www.tcax.org/docs/ass-specs.htm)
- [libass 官方文档](https://github.com/libass/libass)
- [FFmpeg subtitles 滤镜文档](https://ffmpeg.org/ffmpeg-filters.html#subtitles-1)

---

**文档版本：** v1.0  
**最后更新：** 2025-12-06  
**作者：** Cascade AI  
**状态：** ✅ 已验证通过
