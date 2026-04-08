---
name: "vastai-docker"
description: "瀚博半导体 Vastai Docker 安装指南。Invoke when user needs to install and configure Vastai Docker runtime for containerized AI workloads on Vastai hardware."
---

# Vastai Docker 安装指南

Vastai Docker 是容器引擎插件，是 VastStream 软件栈的基础组件，为所有的 AI 推理作业提供瀚博 AI 处理器的容器化支持，使其能够以 Docker 容器的方式运行在瀚博设备之上。

## 简介

Vastai Docker 本质是基于 OCI（Open Container Initiative）标准实现的 Docker Runtime，以插件方式向原生 Docker 提供瀚博 AI 处理器适配功能。

### 工作原理

Vastai Docker 通过 OCI 接口与原生 Docker 对接。在原生 Docker 的 runc 启动容器过程中，会调用 Prestart-hook 对容器进行配置管理。

在 Prestart-hook 中，Vastai Docker 执行以下操作：

1. **设备挂载**: 根据 `VASTAI_VISIBLE_DEVICES` 环境变量，将对应的卡挂载到容器的 namespace，保证设备资源的隔离
2. **Cgroup 配置**: 在 Host 上配置该容器的 device cgroup，确保该容器只可以使用指定的卡，实现对进程的资源限制
3. **Runtime Library 挂载**: 将 Host 上的 Vastai Runtime Library 挂载到容器的 namespace

## 前提条件

### 支持的 Docker 版本

| Docker 版本 | x86_64 | ppc64le | arm64/aarch64 |
|-------------|--------|---------|---------------|
| Docker 18.09 | ✓ | × | ✓ |
| Docker 19.03 | ✓ | × | ✓ |
| Docker 20.10 | ✓ | × | ✓ |

## 安装步骤

### 1. 安装原生 Docker

#### CentOS Docker 安装

**步骤 1**: 卸载旧版本

```bash
sudo yum remove docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine
```

**步骤 2**: 设置 Docker 仓库

```bash
# 使用官方源
sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
```

或使用国内源（推荐）：

```bash
# 使用阿里云源
sudo yum-config-manager \
  --add-repo \
  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**步骤 3**: 安装 Docker

```bash
# 安装最新版本
sudo yum install docker-ce docker-ce-cli containerd.io
```

安装指定版本：

```bash
# 列出可用版本
yum list docker-ce --showduplicates | sort -r

# 安装指定版本
sudo yum install docker-ce-<VERSION_STRING> \
  docker-ce-cli-<VERSION_STRING> \
  containerd.io
```

**步骤 4**: 启动 Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

**步骤 5**: 验证安装

```bash
docker version
```

#### Ubuntu Docker 安装

**步骤 1**: 卸载旧版本

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**步骤 2**: 更新 apt 包索引并安装依赖

```bash
sudo apt-get update
sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common
```

**步骤 3**: 添加 Docker 官方 GPG 密钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

验证密钥：

```bash
sudo apt-key fingerprint 0EBFCD88
```

**步骤 4**: 设置 Docker 仓库

```bash
# 添加稳定版仓库
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

**步骤 5**: 安装 Docker

```bash
# 更新 apt 包索引
sudo apt-get update

# 安装最新版本
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

安装指定版本：

```bash
# 列出可用版本
apt-cache madison docker-ce

# 安装指定版本
sudo apt-get install docker-ce=<VERSION_STRING> \
  docker-ce-cli=<VERSION_STRING> \
  containerd.io
```

**步骤 6**: 验证安装

```bash
# 运行测试容器
sudo docker run hello-world
```

### 2. 安装 Vastai Docker

#### Ubuntu Vastai Docker 安装

**步骤 1**: 获取安装包

```bash
# 解压安装包
tar -xzvf vastai-docker-ubuntu.tar.gz
cd vastai-docker-ubuntu
```

**步骤 2**: 安装

```bash
# 安装 deb 包
sudo dpkg -i vastai-docker_*.deb

# 如果依赖缺失，执行
sudo apt-get install -f
```

#### CentOS Vastai Docker 安装

**步骤 1**: 获取安装包

```bash
# 解压安装包
tar -xzvf vastai-docker-centos.tar.gz
cd vastai-docker-centos
```

**步骤 2**: 安装

```bash
# 安装 rpm 包
sudo rpm -ivh vastai-docker-*.rpm
```

### 3. 使能 Vastai Docker

**步骤 1**: 配置 Docker daemon

编辑 `/etc/docker/daemon.json`，添加 runtime 配置：

```json
{
  "runtimes": {
    "vastai": {
      "path": "/usr/bin/vastai-docker",
      "runtimeArgs": []
    }
  },
  "default-runtime": "vastai"
}
```

**步骤 2**: 重启 Docker 服务

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

**步骤 3**: 验证配置

```bash
# 查看 runtime 列表
docker info | grep -A 5 "Runtimes"

# 应该看到 vastai runtime
```

## PCIe 驱动安装与维护

### 环境准备

- 确保已安装 kernel-devel 和 kernel-headers
- 确保已安装 gcc 和 make

```bash
# Ubuntu
sudo apt-get install linux-headers-$(uname -r) build-essential

# CentOS
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc make
```

### PCIe 驱动安装

#### Ubuntu PCIe 驱动安装

**步骤 1**: 获取驱动包

```bash
tar -xzvf vastai-pcie-driver-ubuntu.tar.gz
cd vastai-pcie-driver-ubuntu
```

**步骤 2**: 安装

```bash
# 安装 deb 包
sudo dpkg -i vastai-pcie-driver_*.deb
```

**步骤 3**: 加载驱动

```bash
sudo modprobe vastai_pcie
```

**步骤 4**: 验证安装

```bash
# 查看驱动版本
cat /sys/module/vastai_pcie/version

# 查看设备
lspci | grep Vastai
```

#### CentOS PCIe 驱动安装

**步骤 1**: 获取驱动包

```bash
tar -xzvf vastai-pcie-driver-centos.tar.gz
cd vastai-pcie-driver-centos
```

**步骤 2**: 安装

```bash
# 安装 rpm 包
sudo rpm -ivh vastai-pcie-driver-*.rpm
```

**步骤 3**: 加载驱动

```bash
sudo modprobe vastai_pcie
```

**步骤 4**: 验证安装

```bash
cat /sys/module/vastai_pcie/version
lspci | grep Vastai
```

### PCIe 驱动升级

#### Ubuntu PCIe 驱动升级

```bash
# 卸载旧版本
sudo modprobe -r vastai_pcie
sudo dpkg -r vastai-pcie-driver

# 安装新版本
sudo dpkg -i vastai-pcie-driver_*.deb
sudo modprobe vastai_pcie
```

#### CentOS PCIe 驱动升级

```bash
# 卸载旧版本
sudo modprobe -r vastai_pcie
sudo rpm -e vastai-pcie-driver

# 安装新版本
sudo rpm -ivh vastai-pcie-driver-*.rpm
sudo modprobe vastai_pcie
```

### PCIe 驱动卸载

#### Ubuntu PCIe 驱动卸载

```bash
sudo modprobe -r vastai_pcie
sudo dpkg -r vastai-pcie-driver
```

#### CentOS PCIe 驱动卸载

```bash
sudo modprobe -r vastai_pcie
sudo rpm -e vastai-pcie-driver
```

## 使用 Vastai Docker

### 运行容器

```bash
# 使用 vastai runtime 运行容器
docker run --runtime=vastai -e VASTAI_VISIBLE_DEVICES=0 \
  -it ubuntu:20.04 /bin/bash
```

### 环境变量

- `VASTAI_VISIBLE_DEVICES`: 指定容器可见的设备

```bash
# 使用单个设备
docker run --runtime=vastai -e VASTAI_VISIBLE_DEVICES=0 ...

# 使用多个设备
docker run --runtime=vastai -e VASTAI_VISIBLE_DEVICES=0,1 ...

# 使用所有设备
docker run --runtime=vastai -e VASTAI_VISIBLE_DEVICES=all ...
```

### 验证设备挂载

在容器内执行：

```bash
# 查看设备
ls -la /dev/vastai*

# 或使用 VAML 工具查询
vastai-smi
```

## 故障排查

### 问题 1: 容器无法启动

**现象**: 容器启动失败，提示 runtime 错误。

**解决方案**:
1. 检查 PCIe 驱动是否加载
2. 检查 Vastai Docker 是否正确安装
3. 查看 Docker 日志

```bash
# 查看驱动状态
lsmod | grep vastai

# 查看 Docker 日志
journalctl -u docker -f
```

### 问题 2: 设备不可见

**现象**: 容器内无法看到 Vastai 设备。

**解决方案**:
1. 检查 `VASTAI_VISIBLE_DEVICES` 环境变量
2. 检查设备权限
3. 重新加载 PCIe 驱动

```bash
# 检查环境变量
docker exec <container_id> env | grep VASTAI

# 检查设备
docker exec <container_id> ls -la /dev/vastai*
```

### 问题 3: 性能异常

**现象**: 容器内推理性能低于预期。

**解决方案**:
1. 检查设备利用率
2. 检查内存带宽
3. 检查是否有其他进程占用设备

```bash
# 查看设备状态
vastai-smi

# 查看进程
vastai-smi p
```

## 相关文档

- Kubernetes 设备插件安装指南
- Vastai Exporter 安装指南
- VAML API 参考手册
