# Ken Burns 转场特效完全指南

## 📖 概述

Ken Burns 接口 (`/v1/video/effects/ken-burns`) 支持 **17 种专业级转场特效**，可以为你的图片视频添加电影级的过渡效果。本文档详细介绍每种特效的使用方法、视觉效果和适用场景。

---

## 🎬 转场特效分类

### 1. 淡入淡出类（Fade）

#### `fade` - 标准淡入淡出
**效果描述**：
- 图片 A 逐渐变透明
- 同时图片 B 逐渐显现
- 两张图片在转场期间叠加显示

**视觉效果**：
```
图片A (100%不透明) → 图片A+B叠加 → 图片B (100%不透明)
```

**适用场景**：
- ✅ 通用场景
- ✅ 温馨、舒缓的内容
- ✅ 风景、旅行视频

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "fade",
    "transition_duration": 0.5
  }
}
```

---

#### `fadeblack` - 黑屏转场 ⭐ 推荐恐怖片
**效果描述**：
- 图片 A 渐暗至黑屏
- 短暂停留在黑屏
- 从黑屏渐亮至图片 B

**视觉效果**：
```
图片A → 渐暗 → 纯黑屏 → 渐亮 → 图片B
```

**适用场景**：
- ✅ **恐怖片**（营造压抑氛围）
- ✅ 悬疑片
- ✅ 严肃、沉重的内容
- ✅ 章节分隔

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "fadeblack",
    "transition_duration": 0.8,
    "magnify_factor": 0.3
  }
}
```

**时长建议**：
- `0.5秒` - 快速转场
- `0.8秒` - **推荐**（营造恐怖氛围）
- `1.2秒` - 慢速、压抑感强

---

#### `fadewhite` - 白屏转场
**效果描述**：
- 图片 A 渐亮至白屏
- 短暂停留在白屏
- 从白屏渐暗至图片 B

**视觉效果**：
```
图片A → 渐亮 → 纯白屏 → 渐暗 → 图片B
```

**适用场景**：
- ✅ 梦境、回忆场景
- ✅ 温馨、浪漫内容
- ✅ 惊吓效果（突然闪白）
- ✅ 时间跳跃

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "fadewhite",
    "transition_duration": 0.6
  }
}
```

---

### 2. 滑动类（Slide）

#### `slideleft` - 向左滑动
**效果描述**：
- 图片 B 从右侧进入
- 图片 A 向左侧退出
- 两张图片同时移动

**视觉效果**：
```
[图片A] → [图片A][图片B] → [图片B]
         ←←←←←←←
```

**适用场景**：
- ✅ 时间线叙事（从右到左）
- ✅ 动感、活力内容
- ✅ 产品展示

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "slideleft",
    "transition_duration": 0.4
  }
}
```

---

#### `slideright` - 向右滑动
**效果描述**：
- 图片 B 从左侧进入
- 图片 A 向右侧退出

**视觉效果**：
```
[图片A] → [图片B][图片A] → [图片B]
         →→→→→→→
```

**适用场景**：
- ✅ 时间线叙事（从左到右）
- ✅ 西方阅读习惯
- ✅ 故事推进

---

#### `slideup` - 向上滑动
**效果描述**：
- 图片 B 从下方进入
- 图片 A 向上方退出

**适用场景**：
- ✅ 上升、进步主题
- ✅ 垂直时间线
- ✅ 层级展示

---

#### `slidedown` - 向下滑动
**效果描述**：
- 图片 B 从上方进入
- 图片 A 向下方退出

**适用场景**：
- ✅ 下降、堕落主题
- ✅ 瀑布流展示
- ✅ 倒叙叙事

---

### 3. 擦除类（Wipe）

#### `wipeleft` - 从左擦除
**效果描述**：
- 图片 B 从左侧开始显现
- 像一条竖线从左向右扫过
- 扫过的地方显示图片 B

**视觉效果**：
```
[A|B] → [A|BB] → [A|BBB] → [BBBB]
```

**适用场景**：
- ✅ 对比展示（前后对比）
- ✅ 揭示效果
- ✅ 复古风格

---

#### `wiperight` - 从右擦除
**效果描述**：
- 图片 B 从右侧开始显现
- 像一条竖线从右向左扫过

---

#### `wipeup` - 从上擦除
**效果描述**：
- 图片 B 从上方开始显现
- 像一条横线从上向下扫过

---

#### `wipedown` - 从下擦除
**效果描述**：
- 图片 B 从下方开始显现
- 像一条横线从下向上扫过

---

### 4. 特殊效果类

#### `circleopen` - 圆形展开
**效果描述**：
- 图片 B 从中心点开始显现
- 像一个圆圈不断扩大
- 最终覆盖整个画面

**视觉效果**：
```
[AAAA] → [A●A] → [A◉A] → [BBBB]
         中心小圆   大圆    完全显示
```

**适用场景**：
- ✅ 聚焦效果
- ✅ 重点突出
- ✅ 望远镜/显微镜视角
- ✅ 复古电影风格

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "circleopen",
    "transition_duration": 0.7
  }
}
```

---

#### `circleclose` - 圆形收缩 ⭐ 推荐诡异效果
**效果描述**：
- 图片 A 从边缘开始消失
- 像一个圆圈不断缩小
- 最终收缩成一个黑点

**视觉效果**：
```
[AAAA] → [B◉B] → [B●B] → [BBBB]
         大圆     小圆    完全显示
```

**适用场景**：
- ✅ **恐怖片**（诡异、压抑）
- ✅ 老电影结束效果
- ✅ 意识消失、昏迷场景
- ✅ 隧道视觉效果

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "circleclose",
    "transition_duration": 0.8,
    "magnify_factor": 0.4
  }
}
```

---

#### `pixelize` - 像素化
**效果描述**：
- 图片 A 逐渐变成大像素块
- 像素块逐渐变成图片 B
- 类似马赛克效果

**视觉效果**：
```
图片A → 大像素块 → 图片B
      (模糊化)
```

**适用场景**：
- ✅ 科技感、数字化主题
- ✅ 游戏风格
- ✅ 现代恐怖片
- ✅ 故障美学

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "pixelize",
    "transition_duration": 0.5
  }
}
```

---

#### `dissolve` - 溶解
**效果描述**：
- 图片 A 随机像素点逐渐变成图片 B
- 类似化学溶解效果
- 比 fade 更有颗粒感

**视觉效果**：
```
图片A → A和B的随机混合 → 图片B
      (颗粒状过渡)
```

**适用场景**：
- ✅ 梦境、幻觉
- ✅ 时间流逝
- ✅ 记忆模糊
- ✅ 艺术片

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "dissolve",
    "transition_duration": 0.6
  }
}
```

---

#### `radial` - 径向擦除
**效果描述**：
- 图片 B 从中心开始以扇形扫过
- 像时钟指针旋转
- 逐渐覆盖整个画面

**视觉效果**：
```
[AAAA] → [A/B] → [AB] → [BBBB]
         扇形    半圆   完全显示
```

**适用场景**：
- ✅ 时间流逝
- ✅ 倒计时效果
- ✅ 旋转、循环主题

---

### 5. 随机模式

#### `random` - 随机转场 ⭐ 推荐惊悚片
**效果描述**：
- 每次转场随机选择上述 16 种效果之一
- 增加不可预测性
- 每次生成视频效果都不同

**适用场景**：
- ✅ **惊悚片**（增加不安感）
- ✅ 实验性内容
- ✅ 快节奏剪辑
- ✅ 避免单调

**配置示例**：
```json
{
  "settings": {
    "transition_effect": "random",
    "transition_duration": 0.6,
    "magnify_factor": 0.5
  }
}
```

**注意事项**：
- ⚠️ 每次生成的转场顺序都不同
- ⚠️ 无法精确控制每个转场点的效果
- ✅ 适合需要多样性的场景

---

## 🎯 场景推荐配置

### 恐怖片配置 1：经典黑屏
```json
{
  "settings": {
    "transition_effect": "fadeblack",
    "transition_duration": 0.8,
    "magnify_factor": 0.3,
    "fps": 24
  }
}
```
**效果**：慢速黑屏转场，营造压抑、恐怖氛围

---

### 恐怖片配置 2：诡异收缩
```json
{
  "settings": {
    "transition_effect": "circleclose",
    "transition_duration": 0.9,
    "magnify_factor": 0.4,
    "fps": 24
  }
}
```
**效果**：圆形收缩，类似意识消失，诡异感强

---

### 恐怖片配置 3：随机惊悚
```json
{
  "settings": {
    "transition_effect": "random",
    "transition_duration": 0.5,
    "magnify_factor": 0.6,
    "fps": 30
  }
}
```
**效果**：快速随机转场，增加紧张感和不可预测性

---

### 温馨风格配置
```json
{
  "settings": {
    "transition_effect": "fade",
    "transition_duration": 1.0,
    "magnify_factor": 0.3,
    "fps": 24
  }
}
```
**效果**：柔和淡入淡出，舒缓温馨

---

### 动感风格配置
```json
{
  "settings": {
    "transition_effect": "slideleft",
    "transition_duration": 0.3,
    "magnify_factor": 0.7,
    "fps": 30
  }
}
```
**效果**：快速滑动，动感十足

---

### 艺术风格配置
```json
{
  "settings": {
    "transition_effect": "dissolve",
    "transition_duration": 1.2,
    "magnify_factor": 0.2,
    "fps": 24
  }
}
```
**效果**：慢速溶解，艺术感强

---

## 📊 参数详解

### `transition_effect`（转场效果）
**类型**：字符串  
**必填**：否  
**默认值**：`null`（无转场）

**可选值**：
```
"fade", "fadeblack", "fadewhite",
"slideleft", "slideright", "slideup", "slidedown",
"wipeleft", "wiperight", "wipeup", "wipedown",
"circleopen", "circleclose",
"pixelize", "dissolve", "radial",
"random"
```

---

### `transition_duration`（转场时长）
**类型**：数字（秒）  
**必填**：否  
**默认值**：`0.5`  
**范围**：`0.1` - `2.0`

**时长建议**：
| 时长 | 效果 | 适用场景 |
|------|------|---------|
| `0.1` - `0.3` | 极快 | 快节奏、紧张 |
| `0.4` - `0.6` | 标准 | 通用 |
| `0.7` - `1.0` | 舒缓 | 恐怖、艺术 |
| `1.1` - `2.0` | 极慢 | 强调、艺术 |

---

### `magnify_factor`（缩放幅度）
**类型**：数字  
**必填**：否  
**默认值**：`0.5`  
**范围**：`0.1` - `1.0`

**效果说明**：
- `0.1` - 几乎不缩放（静态）
- `0.3` - 轻微缩放（优雅）
- `0.5` - 中等缩放（电影感）
- `0.7` - 明显缩放（动感）
- `1.0` - 最大缩放（激进）

---

### `fps`（帧率）
**类型**：整数  
**必填**：否  
**默认值**：`30`  
**范围**：`15` - `60`

**帧率建议**：
- `24fps` - 电影感（推荐恐怖片）
- `30fps` - 流畅标准
- `60fps` - 极致流畅（运动视频）

---

## 💡 最佳实践

### 1. 转场时长与图片显示时间的关系
```
建议：转场时长 ≤ 图片显示时间的 20%
```

**示例**：
- 图片显示 6 秒
- 转场时长建议：`0.6` - `1.2` 秒

---

### 2. 转场效果与内容风格匹配

| 内容风格 | 推荐转场 | 避免使用 |
|---------|---------|---------|
| 恐怖片 | `fadeblack`, `circleclose`, `random` | `fadewhite`, `slide*` |
| 温馨片 | `fade`, `fadewhite`, `dissolve` | `pixelize`, `circleclose` |
| 动作片 | `slide*`, `wipe*`, `random` | `dissolve`, 慢速转场 |
| 艺术片 | `dissolve`, `radial`, 慢速转场 | `slide*`, 快速转场 |

---

### 3. 转场时长与节奏控制

**慢节奏**（恐怖、艺术）：
```json
{
  "transition_duration": 0.8,
  "fps": 24
}
```

**快节奏**（动作、惊悚）：
```json
{
  "transition_duration": 0.3,
  "fps": 30
}
```

---

## 🔧 完整配置示例

### 示例 1：恐怖故事视频
```json
{
  "images": {
    "urls": [...]
  },
  "audio": {
    "url": "http://minio:9000/audio-assets/narration.mp3"
  },
  "background_music": {
    "url": "http://minio:9000/audio-assets/horror_bgm.mp3",
    "volume": 0.2
  },
  "settings": {
    "num_images": 101,
    "magnify_factor": 0.4,
    "transition_effect": "fadeblack",
    "transition_duration": 0.8,
    "fps": 24,
    "auto_subtitle": true,
    "subtitle_language": "en",
    "subtitle_task": "translate"
  },
  "use_nvidia": true
}
```

---

### 示例 2：随机转场惊悚片
```json
{
  "images": {
    "urls": [...]
  },
  "audio": {
    "url": "http://minio:9000/audio-assets/narration.mp3"
  },
  "settings": {
    "num_images": 50,
    "magnify_factor": 0.6,
    "transition_effect": "random",
    "transition_duration": 0.5,
    "fps": 30
  },
  "use_nvidia": true
}
```

---

### 示例 3：温馨旅行视频
```json
{
  "images": {
    "urls": [...]
  },
  "audio": {
    "url": "http://minio:9000/audio-assets/music.mp3"
  },
  "settings": {
    "num_images": 30,
    "magnify_factor": 0.3,
    "transition_effect": "fade",
    "transition_duration": 1.0,
    "fps": 24
  },
  "use_nvidia": true
}
```

---

## ⚠️ 注意事项

1. **转场时长限制**：
   - 最小：`0.1` 秒
   - 最大：`2.0` 秒
   - 超出范围会报错

2. **随机模式的不可预测性**：
   - `random` 模式每次生成的转场顺序都不同
   - 如果需要可重复的效果，请指定具体转场类型

3. **性能考虑**：
   - 复杂转场（如 `pixelize`, `dissolve`）比简单转场（如 `fade`）渲染时间更长
   - 建议启用 `use_nvidia: true` 以加速渲染

4. **转场与字幕的配合**：
   - 转场期间字幕可能被遮挡
   - 建议转场时长不要过长

---

## 📚 相关文档

- [Ken Burns 接口完整文档](./API接口完整文档-第2部分-视频处理.md)
- [视频生成最佳实践](./视频生成最佳实践.md)
- [字幕配置指南](./字幕配置指南.md)

---

## 🎬 总结

Ken Burns 接口提供了 **17 种专业级转场特效**，涵盖了从简单的淡入淡出到复杂的像素化效果。通过合理搭配 `transition_effect`、`transition_duration` 和 `magnify_factor` 参数，你可以创造出符合各种风格的视频作品。

**推荐组合**：
- 恐怖片：`fadeblack` + 0.8秒 + 24fps
- 惊悚片：`random` + 0.5秒 + 30fps
- 温馨片：`fade` + 1.0秒 + 24fps
- 动作片：`slideleft` + 0.3秒 + 30fps
