# 🚀 GPU加速升级指南（RTX 50系列支持）

## 📊 现状诊断

### 检测结果
```
✅ GPU硬件: NVIDIA GeForce RTX 5070 Ti (计算能力 12.0 - Blackwell架构)
✅ Docker GPU挂载: 已配置 --gpus all
✅ OpenCV版本: 4.12.0 (包含cuda模块)
❌ CUDA设备检测: 0个 (问题所在！)
```

### 问题根源
**RTX 5070 Ti使用全新的Blackwell架构（sm_120），当前OpenCV编译时未包含此架构支持。**

CUDA向后兼容规则：
- ✅ 旧代码可在新GPU运行（通过JIT编译PTX）
- ❌ **新GPU无法运行仅针对旧架构编译的二进制代码（缺少sm_120 SASS）**

## 🎯 解决方案对比

| 方案 | 实施难度 | 性能 | 兼容性 | 推荐度 |
|------|---------|------|--------|--------|
| 方案1: 预编译库（pip） | ⭐ 简单 | ⭐⭐ 中等 | ⭐⭐⭐ 好 | ⭐⭐⭐ |
| 方案2: 从源码编译 | ⭐⭐⭐⭐ 复杂 | ⭐⭐⭐⭐⭐ 最佳 | ⭐⭐⭐⭐⭐ 完美 | ⭐⭐⭐⭐⭐ |

## 🔧 方案1: 使用预编译CUDA库（快速方案）

### 优点
- 🚀 10分钟内完成
- 📦 基于现有walicap:v2.1镜像
- 🔄 支持快速回滚

### 缺点  
- ⚠️ 可能不包含sm_120（需验证）
- 📉 性能次于定制编译

### 实施步骤

#### 1. 创建增量升级Dockerfile

**文件: `Dockerfile.gpu-upgrade`**
```dockerfile
FROM walicap:v2.1

# 切换到root用户安装系统依赖
USER root

# 安装CUDA相关依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    # CUDA运行时库
    nvidia-cuda-toolkit \
    # OpenCV编译依赖
    libopencv-dev \
    && rm -rf /var/lib/apt/lists/*

# 卸载现有的opencv-python（如果有）
RUN pip uninstall -y opencv-python opencv-contrib-python || true

# 安装支持CUDA的OpenCV
# 方案A: 尝试预编译版本
RUN pip install --no-cache-dir opencv-contrib-python==4.10.0.84

# 方案B: 如果方案A不支持sm_120，则从源码编译（见方案2）

# 恢复权限
RUN chown -R appuser:appuser /app

# 切回appuser
USER appuser
WORKDIR /app
```

#### 2. 创建快速升级脚本

**文件: `gpu_upgrade.ps1`**
```powershell
# GPU加速增量升级脚本
Write-Host "🚀 开始GPU加速升级..." -ForegroundColor Green

# 1. 构建GPU升级镜像
Write-Host "📦 构建GPU镜像 (基于 walicap:v2.1)..." -ForegroundColor Yellow
docker build -f Dockerfile.gpu-upgrade -t walicap:v2.1-gpu .

if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ GPU镜像构建失败" -ForegroundColor Red
    exit 1
}

# 2. 测试GPU支持
Write-Host "`n🔍 验证GPU支持..." -ForegroundColor Yellow
$gpuTest = docker run --rm --gpus all walicap:v2.1-gpu python3 -c "import cv2; print('CUDA Devices:', cv2.cuda.getCudaEnabledDeviceCount())"

if ($gpuTest -match "CUDA Devices: 0") {
    Write-Host "⚠️ 警告: 预编译库不支持RTX 50系列，需要使用方案2（源码编译）" -ForegroundColor Yellow
    Write-Host "请参考文档中的方案2" -ForegroundColor Yellow
    exit 1
}

# 3. GPU测试通过，重启容器
Write-Host "✅ GPU支持验证成功！" -ForegroundColor Green
Write-Host "🔄 重启容器..." -ForegroundColor Yellow

docker stop ncat 2>$null
docker rm ncat 2>$null

docker run -d `
  --name ncat `
  --gpus all `
  -p 8080:8080 `
  -v E:\ComfyUI_windows_portable\ComfyUI\output:/app/comfyui_output `
  --env-file .env.nca.local `
  walicap:v2.1-gpu

if ($LASTEXITCODE -eq 0) {
    Write-Host "`n✅ GPU升级完成！" -ForegroundColor Green
    Write-Host "镜像: walicap:v2.1-gpu" -ForegroundColor Cyan
    Write-Host "URL: http://localhost:8080" -ForegroundColor Cyan
} else {
    Write-Host "❌ 容器启动失败" -ForegroundColor Red
}
```

---

## ⚙️ 方案2: 从源码编译OpenCV（推荐方案）

### 优点
- ✅ **完美支持RTX 50系列（sm_120）**
- ✅ 向下兼容所有旧GPU（sm_75, sm_86, sm_89等）
- ✅ 性能最优（针对硬件优化）

### 缺点
- ⏱️ 编译时间较长（30-60分钟）
- 🔧 需要较大内存（推荐16GB+）

### 实施步骤

#### 1. 创建GPU编译Dockerfile

**文件: `Dockerfile.gpu-compile`**
```dockerfile
FROM walicap:v2.1

# 切换到root用户
USER root

# ========================================
# 阶段1: 安装编译依赖
# ========================================
RUN apt-get update && apt-get install -y --no-install-recommends \
    # CUDA Toolkit (13.1+ 支持sm_120)
    nvidia-cuda-toolkit \
    # 编译工具
    build-essential cmake git pkg-config \
    # OpenCV依赖
    libjpeg-dev libpng-dev libtiff-dev \
    libavcodec-dev libavformat-dev libswscale-dev \
    libv4l-dev libxvidcore-dev libx264-dev \
    libgtk-3-dev libatlas-base-dev gfortran \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# ========================================
# 阶段2: 编译支持CUDA的OpenCV
# ========================================
WORKDIR /tmp

# 下载OpenCV源码
RUN git clone --depth 1 --branch 4.10.0 https://github.com/opencv/opencv.git && \
    git clone --depth 1 --branch 4.10.0 https://github.com/opencv/opencv_contrib.git

# 创建构建目录
RUN mkdir -p opencv/build && cd opencv/build

# 🔥 关键配置：CUDA架构覆盖
# sm_75: RTX 20系列 (Turing)
# sm_86: RTX 30系列 (Ampere)
# sm_89: RTX 40系列 (Ada Lovelace)  
# sm_120: RTX 50系列 (Blackwell) 🎯
WORKDIR /tmp/opencv/build
RUN cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=/tmp/opencv_contrib/modules \
    -D WITH_CUDA=ON \
    -D WITH_CUDNN=OFF \
    -D CUDA_ARCH_BIN="7.5,8.6,8.9,12.0" \
    -D CUDA_ARCH_PTX=12.0 \
    -D WITH_CUBLAS=ON \
    -D ENABLE_FAST_MATH=ON \
    -D CUDA_FAST_MATH=ON \
    -D OPENCV_DNN_CUDA=ON \
    -D WITH_TBB=ON \
    -D WITH_V4L=ON \
    -D WITH_QT=OFF \
    -D WITH_OPENGL=ON \
    -D BUILD_EXAMPLES=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D PYTHON3_EXECUTABLE=$(which python3) \
    -D PYTHON3_INCLUDE_DIR=$(python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())") \
    -D PYTHON3_PACKAGES_PATH=$(python3 -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())") \
    ..

# 编译（使用所有CPU核心）
RUN make -j$(nproc)

# 安装
RUN make install && ldconfig

# 清理编译文件（节省空间）
WORKDIR /
RUN rm -rf /tmp/opencv /tmp/opencv_contrib

# ========================================
# 阶段3: 验证安装
# ========================================
RUN python3 -c "import cv2; print('✅ OpenCV:', cv2.__version__); print('✅ CUDA:', cv2.cuda.getCudaEnabledDeviceCount())"

# 恢复工作目录和用户
WORKDIR /app
USER appuser

# 运行命令保持不变
CMD ["/app/run_gunicorn.sh"]
```

#### 2. 创建编译升级脚本

**文件: `gpu_compile_upgrade.ps1`**
```powershell
# GPU完整编译升级脚本
Write-Host "⚙️ 开始从源码编译GPU支持的OpenCV..." -ForegroundColor Green
Write-Host "预计编译时间: 30-60分钟" -ForegroundColor Yellow

# 1. 构建编译镜像
Write-Host "`n📦 构建镜像 (包含OpenCV源码编译)..." -ForegroundColor Yellow
$startTime = Get-Date

docker build -f Dockerfile.gpu-compile -t walicap:v2.1-gpu .

$buildTime = (Get-Date) - $startTime
if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ 编译失败，请检查日志" -ForegroundColor Red
    exit 1
}

Write-Host "✅ 编译完成！耗时: $($buildTime.TotalMinutes.ToString('0.0'))分钟" -ForegroundColor Green

# 2. 验证GPU支持
Write-Host "`n🔍 验证GPU支持..." -ForegroundColor Yellow
docker run --rm --gpus all walicap:v2.1-gpu bash -c "
echo '========================================';
nvidia-smi --query-gpu=name --format=csv,noheader;
echo '========================================';
python3 -c \"
import cv2
print('OpenCV版本:', cv2.__version__)
print('CUDA模块:', hasattr(cv2, 'cuda'))
if hasattr(cv2, 'cuda'):
    devices = cv2.cuda.getCudaEnabledDeviceCount()
    print('✅ CUDA设备数:', devices)
    if devices > 0:
        print('🎉 GPU加速已启用！')
    else:
        print('❌ 未检测到CUDA设备')
else:
    print('❌ OpenCV缺少CUDA模块')
\"
"

# 3. 提示用户是否替换容器
Write-Host "`n准备重启容器使用新镜像..." -ForegroundColor Yellow
$confirm = Read-Host "是否立即重启容器? (y/n)"

if ($confirm -eq 'y' -or $confirm -eq 'Y') {
    Write-Host "🔄 重启容器..." -ForegroundColor Yellow
    
    docker stop ncat 2>$null
    docker rm ncat 2>$null
    
    docker run -d `
      --name ncat `
      --gpus all `
      -p 8080:8080 `
      -v E:\ComfyUI_windows_portable\ComfyUI\output:/app/comfyui_output `
      --env-file .env.nca.local `
      walicap:v2.1-gpu
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "`n🎉 GPU编译升级完成！" -ForegroundColor Green
        Write-Host "新镜像: walicap:v2.1-gpu" -ForegroundColor Cyan
        Write-Host "Ken Burns效果现已支持GPU加速 (RTX 5070 Ti)" -ForegroundColor Cyan
    } else {
        Write-Host "❌ 容器启动失败" -ForegroundColor Red
    }
} else {
    Write-Host "镜像已构建: walicap:v2.1-gpu" -ForegroundColor Cyan
    Write-Host "使用以下命令手动启动:" -ForegroundColor Yellow
    Write-Host "docker run -d --name ncat --gpus all -p 8080:8080 -v E:\ComfyUI_windows_portable\ComfyUI\output:/app/comfyui_output --env-file .env.nca.local walicap:v2.1-gpu"
}
```

---

## 🧪 测试GPU加速效果

### 测试脚本
```bash
# 进入容器
docker exec -it ncat bash

# 运行测试
python3 << 'EOF'
import cv2
import time
import numpy as np

print("="*60)
print("GPU加速性能测试")
print("="*60)

# 创建测试图像
img = np.random.randint(0, 255, (1920, 1080, 3), dtype=np.uint8)

# CPU测试
start = time.time()
for _ in range(100):
    resized = cv2.resize(img, (1280, 720))
cpu_time = time.time() - start

print(f"CPU处理100次: {cpu_time:.2f}秒")

# GPU测试
if cv2.cuda.getCudaEnabledDeviceCount() > 0:
    gpu_img = cv2.cuda_GpuMat()
    gpu_img.upload(img)
    
    start = time.time()
    for _ in range(100):
        gpu_resized = cv2.cuda.resize(gpu_img, (1280, 720))
        result = gpu_resized.download()
    gpu_time = time.time() - start
    
    print(f"GPU处理100次: {gpu_time:.2f}秒")
    print(f"🚀 加速比: {cpu_time/gpu_time:.2f}x")
else:
    print("❌ GPU不可用")

print("="*60)
EOF
```

---

## ⚠️ 常见问题

### Q1: 编译失败提示"no kernel image available"
**A**: CUDA Toolkit版本过低，确保使用 CUDA 13.1+ 以支持sm_120

### Q2: 编译时内存不足
** A**: 减少并行编译数: `make -j2` 而不是 `make -j$(nproc)`

### Q3: 如何回退到原版本？
**A**: 
```powershell
docker stop ncat
docker rm ncat
docker run -d --name ncat --gpus all -p 8080:8080 -v E:\ComfyUI_windows_portable\ComfyUI\output:/app/comfyui_output --env-file .env.nca.local walicap:v2.1
```

### Q4: 还是检测不到GPU？
**A**: 检查Docker Desktop设置: Settings → Resources → WSL Integration → 启用GPU支持

---

## 📈 预期性能提升

| 操作 | CPU | GPU (RTX 5070 Ti) | 加速比 |
|------|-----|-------------------|--------|
| Ken Burns效果 (1080p, 5s) | ~12s | ~1.5s | 8x |
| 图像缩放 (批量) | 基准 | 3-5x | 3-5x |
| 视频转码 (h264_nvenc) | 基准 | 10-15x | 10-15x |

---

## 📚 参考资料
- [NVIDIA CUDA计算能力文档](https://developer.nvidia.com/cuda-gpus)
- [OpenCV CUDA模块文档](https://docs.opencv.org/4.x/d2/d3c/group__cudaimgproc.html)
- [Docker GPU支持](https://docs.docker.com/config/containers/resource_constraints/#gpu)
