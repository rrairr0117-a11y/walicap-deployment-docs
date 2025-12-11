# 瓦力聚合剪辑系统 - 部署文档

<div align="center">

![瓦力AI](https://img.shields.io/badge/瓦力AI-rrairr-blue)
![Docker](https://img.shields.io/badge/Docker-Ready-brightgreen)
![License](https://img.shields.io/badge/License-Commercial-orange)
![Version](https://img.shields.io/badge/Version-v2.1-blue)

**强大的视频剪辑 API 系统 | 支持字幕、音频、视频处理 | GPU 加速 | Docker 一键部署**

[购买授权](https://rrairr.cn) · [API 文档](#-api-文档) · [快速开始](#-快速开始) · [技术支持](https://rrairr.cn)

</div>

---

## 📖 项目简介

瓦力聚合剪辑系统是一个功能强大的视频处理 API 平台，提供完整的视频剪辑、字幕处理、音频合成等功能。

> **📦 本仓库包含**：完整的 Docker 部署配置 + 环境变量示例 + API 接口文档  
> **🚫 不包含**：源代码（需购买授权获取）  
> **✅ 开箱即用**：下载后配置环境变量，一键启动

---

## 📂 仓库文件说明

| 文件/目录 | 说明 | 必需 |
|----------|------|------|
| `docker-compose.yml` | Docker 部署配置（GPU/CPU 版本可切换） | ✅ |
| `.env.nca.local.example` | **环境变量配置示例**（复制后填入真实配置） | ✅ |
| `README.md` | 项目说明和快速开始指南 | ✅ |
| `API接口*.md` | 完整的 API 接口文档（7个文件） | 📖 |
| `ASS字幕*.md` | ASS 字幕技术文档（2个文件） | 📖 |
| `音画同步接口*.md` | 音画同步详细说明 | 📖 |
| `功能完整性检查.md` | 系统功能清单 | 📖 |

**⚠️ 重要提示**：  
- 首次使用必须配置 `.env.nca.local` 文件（从 `.env.nca.local.example` 复制）
- 需要从 [https://rrairr.cn](https://rrairr.cn) 获取授权信息

---

### ✨ 核心特性

- 🎬 **视频处理** - 剪辑、合成、特效、水印、横幅
- 📝 **字幕系统** - 自动生成、翻译、ASS 高级字幕、背景框
- 🎵 **音频处理** - 转写、拼接、TTS、音画同步
- 🖼️ **图片处理** - 文字转图、网页截图、图片转视频
- 🔄 **格式转换** - 视频/音频格式转换、媒体信息提取
- ⚡ **GPU 加速** - 支持 NVIDIA GPU 加速处理
- 🐳 **Docker 部署** - 一键启动，开箱即用
- 🔐 **授权保护** - 单设备授权，安全可靠

---

## 🚀 快速开始

> 💡 **提示**：支持在线安装和离线安装两种方式

### 前置要求

- Docker 20.10+
- Docker Compose 2.0+
- 4GB+ 内存
- （可选）NVIDIA GPU + nvidia-docker

### 📦 安装方式选择

#### 方式一：在线安装（推荐）

适用于可以访问阿里云镜像仓库的环境。

#### 方式二：离线安装

适用于网络受限或无法访问镜像仓库的环境。

👉 **离线安装请查看**：[镜像离线安装指南.md](./镜像离线安装指南.md)

---

### 步骤 1: 获取 Docker 镜像

**在线方式**：拉取镜像

```bash
# CPU 版本（v2.0）
docker pull crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.0

# GPU 版本（v2.1cuda - 推荐）
docker pull crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.1cuda
```

**离线方式**：加载镜像文件

```bash
# 从云盘下载镜像文件后加载
docker load -i walicap-v2.1cuda-docker-image.tar
```

📥 **镜像文件下载地址**：
- 夸克网盘：`https://pan.quark.cn/s/4a6aab126d35` 提取码：`/~67b139XHH7~:/`
- 文件大小：9.35 GB
- MD5 校验：`a75a446660622cfb3d7bba9dcde1f51a`

> 💡 **重要提示**：下载后请验证 MD5 值，确保文件完整。详细步骤请查看 [镜像离线安装指南](./镜像离线安装指南.md)

### 步骤 2: 下载部署文件

```bash
# 克隆本仓库
git clone <your-repo-url>
cd <repo-name>

# 或直接下载以下文件：
# - docker-compose.yml
# - .env.nca.local（环境配置）
```

### 步骤 3: 配置环境变量

复制并编辑环境配置文件：

```bash
cp .env.nca.local.example .env.nca.local
```

**必填配置**：

```bash
# API 认证
API_KEY=你的API密钥

# 授权信息（从 https://rrairr.cn 获取）
LICENSE_API_SECRET=sk-你的授权密钥
LICENSE_PRODUCT_ID=65
LICENSE_KEY=LIC_你的授权码
LICENSE_USERNAME=你的用户名
```

### 步骤 4: 修改 docker-compose.yml

根据你拉取的镜像版本，修改 `docker-compose.yml` 中的镜像地址：

```yaml
services:
  ncat:
    # CPU 版本
    image: crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.0
    
    # 或 GPU 版本（推荐）
    # image: crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.1cuda
```

### 步骤 5: 启动服务

```bash
# 启动所有服务（NCA + MinIO）
docker-compose up -d

# 查看日志
docker-compose logs -f ncat

# 检查服务状态
docker-compose ps
```

### 步骤 6: 验证部署

```bash
# 测试 API
curl -X GET "http://localhost:8080/v1/toolkit/test" \
  -H "x-api-key: 你的API_KEY"

# 预期返回
{
  "message": "success",
  "build_number": 225
}
```

---

## 🐳 Docker 镜像说明

### 可用版本

| 版本 | 镜像地址 | 大小 | 说明 |
|------|---------|------|------|
| **v2.1cuda** | `crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.1cuda` | ~5.4GB | GPU 加速版本（推荐） |
| v2.0 | `crpi-gpb2zeuxu5n5h1lc.cn-chengdu.personal.cr.aliyuncs.com/rrairr/walicap:v2.0` | ~3.2GB | CPU 版本 |

### 版本特性对比

| 特性 | v2.0 (CPU) | v2.1cuda (GPU) |
|------|-----------|----------------|
| 基础视频处理 | ✅ | ✅ |
| 字幕生成 | ✅ | ✅ |
| 音频转写 | ✅ (慢) | ✅ (快 10x) |
| GPU 加速 | ❌ | ✅ |
| CUDA 支持 | ❌ | ✅ |
| 推荐场景 | 测试/轻量使用 | 生产环境 |

---

## 📚 API 文档

### 完整文档列表

- **[API 接口统一规范](./API接口统一规范.md)** - 接口调用规范和认证方式
- **[API 接口汇总索引](./API接口汇总索引.md)** - 所有接口快速索引
- **[API 接口完整文档 - 第1部分](./API接口完整文档-第1部分-字幕视频.md)** - 字幕和视频处理
- **[API 接口完整文档 - 第2部分](./API接口完整文档-第2部分-音频图片格式转换.md)** - 音频图片格式转换
- **[API 接口完整文档 - 第3部分](./API接口完整文档-第3部分-高级功能与下载.md)** - 高级功能与下载
- **[数据源与路径要求](./API接口数据源与路径要求补充说明.md)** - 文件路径和数据源说明
- **[音画同步接口说明](./音画同步接口完整请求体说明.md)** - 音画同步详细文档

### 技术文档

- **[ASS 字幕背景框实现](./ASS字幕背景框实现技术文档.md)** - ASS 字幕高级特效
- **[ASS 字幕背景框与发光效果](./ASS字幕背景框与发光效果实现文档.md)** - 字幕特效详解
- **[功能完整性检查](./功能完整性检查.md)** - 系统功能清单

### 快速示例

#### 1. 视频添加字幕

```bash
curl -X POST "http://localhost:8080/v1/video/caption" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_url": "https://example.com/video.mp4",
    "subtitle_url": "https://example.com/subtitle.srt",
    "cloud_upload": true
  }'
```

#### 2. 音频转写

```bash
curl -X POST "http://localhost:8080/v1/media/transcribe" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "media_url": "https://example.com/audio.mp3",
    "task": "transcribe",
    "language": "zh"
  }'
```

#### 3. 视频拼接

```bash
curl -X POST "http://localhost:8080/v1/video/concatenate" \
  -H "x-api-key: 你的API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "video_urls": [
      "https://example.com/video1.mp4",
      "https://example.com/video2.mp4"
    ],
    "cloud_upload": true
  }'
```

---

## 🔧 配置说明

### MinIO 对象存储

系统集成了 MinIO 作为本地对象存储：

- **Web 控制台**: http://localhost:9001
- **默认账号**: minioadmin / minioadmin123
- **S3 API**: http://localhost:9000

### 环境变量配置

详细配置说明请查看 `.env.nca.local` 文件，主要配置项：

```bash
# 基础配置
API_KEY=你的API密钥
LOCAL_STORAGE_PATH=/tmp

# 存储配置
STORAGE_PROVIDER=S3
S3_ENDPOINT_URL=http://minio:9000
S3_BUCKET_NAME=nca-toolkit-local

# 转写引擎
TRANSCRIBE_ENGINE=whisper  # 或 funasr

# 翻译服务
TRANSLATE_SERVICE=youdao
YOUDAO_APP_ID=你的AppID
YOUDAO_APP_KEY=你的AppKey

# 授权验证
LICENSE_API_SECRET=你的授权密钥
LICENSE_PRODUCT_ID=65
LICENSE_KEY=你的授权码
```

---

## 🔐 授权说明

### 获取授权

1. 访问 [https://rrairr.cn](https://rrairr.cn)
2. 购买「瓦力聚合剪辑系统」授权
3. 获取授权码和 API 密钥
4. 配置到 `.env.nca.local` 文件

### 授权特性

- ✅ 单设备授权（基于 Docker 容器 ID）
- ✅ 自动验证授权状态
- ✅ 显示剩余天数和到期时间
- ✅ 支持设备解绑和重新绑定

### 授权验证

启动时会自动验证授权，显示如下信息：

```
╔══════════════════════════════════════════════════════════════╗
║ 瓦力AI rrairr · 瓦力聚合剪辑系统                              ║
║ 购买: https://rrairr.cn                                      ║
║ 授权: 已激活 · 剩余 365 天 · 到期 2025-12-09               ║
║ 用户: user@example.com                                       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🛠️ 系统要求

### 最低配置

- **CPU**: 2 核心
- **内存**: 4GB RAM
- **存储**: 20GB
- **系统**: Linux / Windows / macOS
- **Docker**: 20.10+

### 推荐配置（GPU 版本）

- **CPU**: 4 核心+
- **内存**: 8GB RAM+
- **GPU**: NVIDIA GPU（支持 CUDA 11.8+）
- **存储**: 50GB+ SSD
- **nvidia-docker**: 已安装

---

## 📊 性能参考

| 任务类型 | CPU 版本 | GPU 版本 | 加速比 |
|---------|---------|---------|--------|
| 音频转写 (10分钟) | ~15分钟 | ~1.5分钟 | 10x |
| 视频字幕生成 | ~20分钟 | ~2分钟 | 10x |
| 视频格式转换 | ~5分钟 | ~2分钟 | 2.5x |
| 图片转视频 | ~3分钟 | ~1分钟 | 3x |

---

## 🐛 故障排除

### 常见问题

**1. 授权验证失败**
```bash
# 检查授权配置
cat .env.nca.local | grep LICENSE

# 查看容器日志
docker-compose logs ncat | grep -i license
```

**2. 端口被占用**
```bash
# 检查端口占用
netstat -ano | findstr :8080

# 修改 docker-compose.yml 中的端口映射
ports:
  - "8081:8080"  # 改用 8081 端口
```

**3. GPU 不可用**
```bash
# 检查 nvidia-docker
docker run --rm --gpus all nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi

# 查看 GPU 状态
nvidia-smi
```

**4. MinIO 连接失败**
```bash
# 检查 MinIO 服务
docker-compose ps minio

# 查看 MinIO 日志
docker-compose logs minio
```

---

## 📞 技术支持

- **官方网站**: [https://rrairr.cn](https://rrairr.cn)
- **购买授权**: 访问官网联系客服
- **技术支持**: 提供工单系统和在线客服
- **文档更新**: 持续更新 API 文档和使用指南

---

## 📄 许可证

本项目采用商业授权模式：

- ✅ 购买授权后可商用
- ✅ 提供技术支持和更新
- ✅ 单设备授权保护
- ❌ 未授权禁止使用

---

## 🎯 更新日志

### v2.1cuda (2025-12-09)
- ✨ 新增 GPU 加速支持
- ✨ 优化音频转写性能（10x 提升）
- ✨ 新增 ASS 字幕背景框功能
- 🐛 修复视频合成内存泄漏
- 📝 完善 API 文档

### v2.0 (2025-12-06)
- 🎉 首个稳定版本发布
- ✨ 完整的视频剪辑功能
- ✨ 字幕生成和翻译
- ✨ 音频处理和 TTS
- ✨ MinIO 对象存储集成

---

<div align="center">

**Copyright © 2025 瓦力AI rrairr**

[购买授权](https://rrairr.cn) · [查看文档](#-api-文档) · [技术支持](https://rrairr.cn)

</div>
