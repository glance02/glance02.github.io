---
layout:     post
title:      "WSL2 + PyTorch CUDA 环境配置踩坑记录"
subtitle:   "WSL2 下 PyTorch 调用 GPU 时的完整排坑过程"
date:       2026-03-15 12:00:00
author:     "glance"
header-img: "img/post-bg-2015.jpg"
tags:
    - 深度学习
---
## 一、问题现象

**环境**：WSL2 (Ubuntu) + RTX 4070 Ti + Conda (vfm环境)

**症状**：
- `nvidia-smi` 正常显示显卡信息（Driver Version: 591.74）
- `ls -l /dev/nvidia*` 报错（No such file or directory）——**这是WSL2的正常现象**
- PyTorch 测试：`torch.cuda.is_available()` 返回 `False`
- `torch.version.cuda` 显示 `11.3`（说明安装了CUDA版本的PyTorch，但运行时找不到驱动）

---

## 二、根本原因

### 1. WSL2 的 GPU 架构与传统 Linux 不同

| 特性 | 传统 Linux | WSL2 |
|------|-----------|------|
| 设备文件 | `/dev/nvidia0`, `/dev/nvidiactl` | **不存在**（使用 `/dev/dxg` 透传） |
| NVIDIA驱动 | 安装在 Linux 内核中 | 安装在 **Windows 主机** 上 |
| CUDA驱动库 | `/usr/lib/x86_64-linux-gnu/libcuda.so` | `/usr/lib/wsl/lib/libcuda.so`（从Windows挂载） |
| 所需组件 | 驱动 + CUDA Toolkit | **仅需 CUDA Toolkit**（工具包） |

### 2. 具体问题定位

PyTorch 调用 GPU 需要：
1. **CUDA Toolkit**（`nvcc` 编译器及运行时库）→ **缺失**
2. **libcuda.so**（NVIDIA驱动运行时库）→ **已存在但不在系统库路径中**
3. **环境变量**（`CUDA_HOME`, `LD_LIBRARY_PATH`）→ **未配置**

---

## 三、完整解决过程

### Step 1: 安装 CUDA Toolkit 11.3（与PyTorch版本匹配）

**注意**：WSL2 内**不要**安装 NVIDIA 驱动（不要选driver选项），驱动由 Windows 管理。

```bash
# 添加 NVIDIA 官方仓库（WSL专用）
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update

# 安装 CUDA Toolkit 11.3（PyTorch 1.11.0 对应 cu113）
sudo apt install cuda-toolkit-11-3 -y
```

**安装位置**：`/usr/local/cuda-11.3/`（而非 `/usr/lib/cuda`）

### Step 2: 配置环境变量

```bash
# 添加到 ~/.bashrc
echo 'export PATH=/usr/local/cuda-11.3/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/lib/wsl/lib:/usr/local/cuda-11.3/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export CUDA_HOME=/usr/local/cuda-11.3' >> ~/.bashrc

source ~/.bashrc
```

**关键点**：
- `LD_LIBRARY_PATH` **必须包含 `/usr/lib/wsl/lib`**（存放 `libcuda.so.1`）
- `CUDA_HOME` 必须设置（PyTorch 通过它定位 CUDA）

### Step 3: 更新系统库缓存

让系统动态链接器找到 WSL 的 GPU 驱动库：

```bash
# 创建 ld.so 配置
echo '/usr/lib/wsl/lib' | sudo tee /etc/ld.so.conf.d/wsl-cuda.conf
echo '/usr/local/cuda-11.3/lib64' | sudo tee /etc/ld.so.conf.d/cuda-11-3.conf

# 更新缓存
sudo ldconfig

# 验证 libcuda 是否被识别
ldconfig -p | grep libcuda
# 应输出：libcuda.so.1 (libc6,x86-64) => /usr/lib/wsl/lib/libcuda.so.1
```

### Step 4: Conda 环境持久化（可选但推荐）

防止在 VS Code/Jupyter 中启动时环境变量丢失：

```bash
conda activate vfm
conda env config vars set CUDA_HOME=/usr/local/cuda-11.3
conda env config vars set LD_LIBRARY_PATH=/usr/lib/wsl/lib:/usr/local/cuda-11.3/lib64

# 重新激活生效
conda deactivate && conda activate vfm
```

---

## 四、验证与测试

```python
import torch

print(f"PyTorch版本: {torch.__version__}")
print(f"CUDA可用: {torch.cuda.is_available()}")  # 应输出 True
print(f"当前设备: {torch.cuda.get_device_name(0)}")  # 应输出 NVIDIA GeForce RTX 4070 Ti
print(f"CUDA版本: {torch.version.cuda}")  # 应输出 11.3
```

**命令行验证**：
```bash
nvcc --version  # 应显示 Cuda compilation tools, release 11.3
which nvcc      # 应输出 /usr/local/cuda-11.3/bin/nvcc
```

---

## 五、踩坑记录

### ❌ 坑1：寻找 `/dev/nvidia*`
**错误认知**：以为必须出现 `/dev/nvidia0` 才代表 GPU 可用。  
**事实**：WSL2 使用 `/dev/dxg` 和 `/usr/lib/wsl/lib/libcuda.so` 机制，**不应该存在** `/dev/nvidia*`。

### ❌ 坑2：路径错误
**错误配置**：初期误以为 apt 安装的 CUDA 在 `/usr/lib/cuda`，配置了错误路径。  
**正确路径**：`/usr/local/cuda-11.3/`（与手动安装位置一致）。

### ❌ 坑3：缺少 `libcuda.so` 路径
**现象**：`nvcc` 已可用，但 PyTorch 仍报 `CUDA available: False`。  
**原因**：`libcuda.so` 位于 `/usr/lib/wsl/lib`（WSL2 从 Windows 挂载的特殊目录），但系统库缓存未包含此路径。  
**解决**：必须通过 `ld.so.conf.d` 配置或 `LD_LIBRARY_PATH` 指定。

### ❌ 坑4：重复安装驱动
**风险**：如果在 WSL2 内运行了 `.run` 文件并勾选了 Driver，会安装 Linux 版驱动，与 Windows 驱动冲突。  
**原则**：WSL2 内**只安装 CUDA Toolkit**，不安装 Driver。

---

## 六、核心要点总结

1. **WSL2 GPU 驱动在 Windows**，Linux 内只需 CUDA Toolkit
2. **必须暴露 `/usr/lib/wsl/lib`** 给 PyTorch（通过 `LD_LIBRARY_PATH` 或 `ldconfig`）
3. **版本匹配**：PyTorch 1.11.0+cu113 → CUDA 11.3 Toolkit
4. **环境变量三件套**：`PATH`（nvcc）、`CUDA_HOME`（PyTorch找CUDA）、`LD_LIBRARY_PATH`（找libcuda.so）