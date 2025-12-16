<div align="center">

# 瓦力聚合剪辑系统

![Docs](https://img.shields.io/badge/Docs-Ready-brightgreen)
![Docker](https://img.shields.io/badge/Docker-Ready-blue)
![Version](https://img.shields.io/badge/Version-v2.1-blue)

**自动化视频处理，解放生产力**

把剪辑从“手工操作”升级成“可复用的工作流”：用 API 把素材跑成成片。

剪辑流水线参考：**转写 -> 双语字幕 -> 音画同步 -> 合成与烧录 -> 云端交付**。

[![3步上手](https://img.shields.io/badge/3%20%E6%AD%A5%E4%B8%8A%E6%89%8B-Quickstart-2ea44f?style=for-the-badge)](#quickstart)
[![API索引](https://img.shields.io/badge/%E6%9F%A5%E7%9C%8B%20API%20%E7%B4%A2%E5%BC%95-Docs-1f6feb?style=for-the-badge)](./user-docs/API接口汇总索引.md)
[![n8n模板](https://img.shields.io/badge/%E4%B8%8B%E8%BD%BD%20n8n%20%E6%A8%A1%E6%9D%BF-Workflow-f59e0b?style=for-the-badge)](./user-docs/瓦力视频系统.json)

</div>

---

## 适合谁使用

- **内容团队 / 短视频矩阵**：批量产出、版本迭代、统一字幕样式
- **出海团队 / 双语内容**：双语字幕稳定分行显示，避免遮挡，风格可控
- **教育 / 课程 / 播客**：音画同步、字幕生成、一键视频化
- **研发团队**：把剪辑能力嵌入产品，用 API 形成可组合的视频服务

---

<a name="why"></a>

## 为什么是瓦力

- **更少人工**：把重复剪辑动作变成一次配置、无限复用
- **更高一致性**：同一套参数、同一套流程，批量跑出来的结果一致可复现
- **更易集成**：API 可直接对接你的后端、脚本或 n8n 工作流
- **更快交付**：任务化处理 + 结果回传，适合规模化内容生产

| 你现在可能在做 | 用瓦力之后可以做成 |
|---|---|
| 手工剪辑：导入、转写、校对、翻译、排版、烧录 | 自动化：提交素材 -> 跑完链路 -> 拿到成片链接 |
| 一条条视频重复操作，容易出错 | 一套模板参数复用，批量生成，结果一致 |
| 多工具拼装，维护成本高 | 能力集中在 API 层，链路可观测可复现 |

---

<a name="highlights"></a>

## 功能亮点（强大之处）

- **字幕能力（从生成到成片）**
  - 自动转写生成字幕
  - 双语字幕：原文/译文分行显示，样式可控，避免遮挡
  - ASS 高级字幕：背景框、发光等效果
- **音画同步（让节奏“对上”）**
  - 音频与视频对齐
  - 静音检测与处理（用于切段、卡点、节奏优化）
- **合成与批量生产（把素材变成成片）**
  - 合并音频与视频，按需烧录字幕
  - 循环短视频匹配长音频（背景视频/lofi/素材轮播）
  - 多图+音频生成视频、常见格式转换、裁剪拼接、信息提取

---

## 典型场景（你可以直接照搬）

- **短视频矩阵批量生产**
  - 音频转写 -> 字幕生成/翻译 -> 烧录字幕 -> 输出到云端
- **海外/双语内容**
  - 中文字幕 + 英文/日文译文双行显示，避免遮挡，样式统一
- **播客/课程一键视频化**
  - 多图/网页截图/背景视频 + 旁白音频 -> 成品视频
- **BGM 长音频配背景循环视频**
  - 短视频素材循环到目标时长 -> 合并长音频 -> 输出 MP4

---

## 你会在这个仓库里得到什么

- **部署所需文件与示例配置**：`user-docs/docker-compose.yml`、`.env.nca.local.example`
- **完整 API 文档**：按主题拆分，便于查阅与复制请求体
- **n8n 工作流模板**：可直接导入 n8n，快速搭建调用链路

说明：本仓库面向“部署与使用”，不承载业务源码。

---

<a name="quickstart"></a>

## 快速开始

所有文档与部署文件都在 `user-docs/` 目录下，建议按以下 **3 步** 使用：

```bash
git clone <your-repo-url>
cd <repo-name>/user-docs

cp .env.nca.local.example .env.nca.local
# 按需编辑 .env.nca.local

docker-compose up -d
```

常用检查：

```bash
docker-compose ps
docker-compose logs -f ncat
```

接口连通性测试（示例）：

```bash
curl -X GET "http://localhost:8080/v1/toolkit/test" \
  -H "x-api-key: 你的API_KEY"
```

---

<a name="try"></a>

## 快速体验（可复制示例）

下面给你 3 个最能体现“瓦力剪辑强大之处”的接口示例，复制即可跑通（把 URL 和 API_KEY 换成你的）。

### 1) 自动生成字幕（含可选翻译/双语）

```bash
curl -X POST "http://localhost:8080/v1/video/caption" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://example.com/video.mp4",
    "cloud_upload": true
  }'
```

### 2) 音画同步（适合配音/节奏对齐）

```bash
curl -X POST "http://localhost:8080/v1/video/sync-audio-video" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://example.com/video.mp4",
    "audio_url": "https://example.com/voice.mp3",
    "cloud_upload": true
  }'
```

### 3) 循环短视频匹配长音频（背景视频/lofi）

```bash
curl -X POST "http://localhost:8080/v1/video/loop-to-audio" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://example.com/short_clip.mp4",
    "audio_url": "https://example.com/long_music.mp3",
    "audio_duration": 600
  }'
```

---

## 接口速览（最常用的一组）

| 场景 | 路径 | 说明 |
|---|---|---|
| 字幕生成 | `/v1/video/caption` | 转写并生成字幕，按需支持双语字幕 |
| 音画同步 | `/v1/video/sync-audio-video` | 让配音/旁白与画面更贴合 |
| 循环视频配长音频 | `/v1/video/loop-to-audio` | 短视频循环到目标时长并合成音频 |
| 视频音频合并 | `/v1/video/merge-audio` | 合成音频轨道，按需烧入字幕 |
| 视频拼接 | `/v1/video/concatenate` | 多段视频合并成一条 |
| 音频转写 | `/v1/media/transcribe` | 音频转文字，便于字幕/翻译 |

---

<a name="docs"></a>

## 文档导航

- **文档首页（建议从这里开始）**
  - [user-docs/README.md](./user-docs/README.md)：部署说明、版本说明、常见问题
- **API 文档与规范**
  - [API接口统一规范](./user-docs/API接口统一规范.md)
  - [API接口汇总索引](./user-docs/API接口汇总索引.md)
  - [API接口完整文档-第1部分-字幕视频](./user-docs/API接口完整文档-第1部分-字幕视频.md)
  - [API接口完整文档-第2部分-音频图片格式转换](./user-docs/API接口完整文档-第2部分-音频图片格式转换.md)
  - [API接口完整文档-第3部分-高级功能与下载](./user-docs/API接口完整文档-第3部分-高级功能与下载.md)
  - [API接口数据源与路径要求补充说明](./user-docs/API接口数据源与路径要求补充说明.md)
  - [音画同步接口完整请求体说明](./user-docs/音画同步接口完整请求体说明.md)
- **ASS 技术文档**
  - [ASS 字幕背景框实现技术文档](./user-docs/ASS%20字幕背景框实现技术文档.md)
  - [ASS字幕背景框与发光效果实现文档](./user-docs/ASS字幕背景框与发光效果实现文档.md)
- **工作流模板（n8n）**
  - [瓦力视频系统.json](./user-docs/瓦力视频系统.json)

---

## 常见问题

### 1) 为什么 GitHub 首页看不到文档？

文档都在 `user-docs/` 目录下：

- 直接进入：[`user-docs/`](./user-docs/)
- 从文档首页开始：[`user-docs/README.md`](./user-docs/README.md)

### 2) `.env.nca.local` 可以直接提交到仓库吗？

建议只提交示例文件：`.env.nca.local.example`。

### 3) 是否支持 GPU？

支持。具体镜像与部署方式以 `user-docs/README.md` 为准。

### 4) 我该从哪里开始看文档？

如果你第一次接入，建议顺序：

- [API接口统一规范](./user-docs/API接口统一规范.md)
- [API接口汇总索引](./user-docs/API接口汇总索引.md)
- 按你的场景选择对应接口文档（第1/2/3部分）
