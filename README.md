<div align="center">

# 瓦力聚合剪辑系统

![Docs](https://img.shields.io/badge/Docs-Ready-brightgreen)
![Docker](https://img.shields.io/badge/Docker-Ready-blue)
![Version](https://img.shields.io/badge/Version-v2.1-blue)

面向自动化工作流的音频/视频/字幕处理 API：字幕生成与翻译、音画同步、视频剪辑与合成、图片视频化、ASS 高级字幕等。

[文档入口](#-文档导航) · [快速开始](#-快速开始) · [n8n 工作流模板](./user-docs/瓦力视频系统.json)

</div>

---

## 你会在这个仓库里得到什么

- **部署所需文件与示例配置**：`user-docs/docker-compose.yml`、`.env.nca.local.example`
- **完整 API 文档**：按主题拆分，便于查阅与复制请求体
- **n8n 工作流模板**：可直接导入 n8n，快速搭建调用链路

说明：本仓库面向“部署与使用”，不承载业务源码。

---

## 快速开始

所有文档与部署文件都在 `user-docs/` 目录下，建议按以下方式使用：

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

---

## 文档导航

- **文档首页（建议从这里开始）**
  - `user-docs/README.md`：部署说明、版本说明、常见问题
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
