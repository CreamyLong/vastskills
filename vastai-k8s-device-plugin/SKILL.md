---
name: "vastai-k8s-device-plugin"
description: "瀚博半导体 Kubernetes 设备插件安装与配置指南。Invoke when user asks about installing, configuring, or using Vastai Device Plugin for Kubernetes cluster to manage Vastai AI accelerators."
---

# Vastai Kubernetes Device Plugin

Vastai Device Plugin 是一个 Kubernetes Daemonset，负责在 K8s 集群中的每个节点上运行容器，以管理和监控瀚博设备。

## 功能特性

- 公开集群中各个节点上的瀚博设备数量
- 跟踪瀚博设备的健康状况
- 在 K8s 集群中运行启用瀚博设备的容器
- 支持整卡分配和 Time Slicing 分配两种资源分配策略

## 环境要求

- 已安装 PCIe 驱动 (版本 >= 0.1)
- 已安装 Vastai Docker (版本 >= 0.1)
- 已安装 K8s (版本 >= 1.16)

## 安装前准备

### 获取软件包

| 资源包 | 说明 |
|--------|------|
| vastai-device-plugin-{arch}-{version}.tar.gz | Vastai Device Plugin 镜像包 |
| vastai-device-plugin.yaml | Vastai Device Plugin 配置文件 |
| none-device-config.yaml | Time Slicing 配置文件 |
| vastai-va-test.yaml | Pod 资源配置文件 |

## 配置低级运行时

### Docker 配置

1. 创建 daemon.json 文件
```bash
sudo touch /etc/docker/daemon.json
```

2. 写入配置内容
```json
{
    "default-runtime": "vastai",
    "runtimes": {
        "vastai": {
            "path": "/usr/bin/vastai-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

3. 重启 Docker
```bash
sudo systemctl restart docker
```

### Containerd 配置

1. 添加配置至 /etc/containerd/config.toml 文件末尾
```toml
version = 2
[plugins]
[plugins."io.containerd.grpc.v1.cri"]
[plugins."io.containerd.grpc.v1.cri".containerd]
default_runtime_name = "vastai"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.vastai]
privileged_without_host_devices = false
runtime_engine = ""
runtime_root = ""
runtime_type = "io.containerd.runc.v2"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.vastai.options]
BinaryName = "/usr/bin/vastai-container-runtime"
```

2. 重启 Containerd
```bash
sudo systemctl restart containerd
```

## 安装 Vastai Device Plugin

### 整卡分配

1. 在指定 K8s 节点创建标签
```bash
kubectl label nodes {nodeName} vastai-device=vastai
```

2. 创建命名空间
```bash
kubectl create ns vastai-plugin
```

3. 创建 vastai-device-plugin.yaml 配置文件并部署 DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: vastai-device-plugin-daemonset
  namespace: vastai-plugin
spec:
  selector:
    matchLabels:
      name: vastai-device-plugin-ds
  template:
    metadata:
      labels:
        name: vastai-device-plugin-ds
    spec:
      nodeSelector:
        vastai-device: vastai
      containers:
      - name: vastai-device-plugin
        image: vastai-device-plugin:latest
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
        - name: device-plugin
          mountPath: /var/lib/kubelet/device-plugins
      volumes:
      - name: device-plugin
        hostPath:
          path: /var/lib/kubelet/device-plugins
```

### Time Slicing 分配

Time Slicing 允许将单个设备资源分配给多个进程，提高资源利用率。

1. 配置 none-device-config.yaml
```yaml
sharing:
  timeSlicing:
    renameByDefault: false
    resources:
    - name: vastai.com/gpu
      replicas: 4
```

2. 应用配置
```bash
kubectl apply -f none-device-config.yaml
```

## 配置设备插件

### 命令行标志/环境变量

| 参数 | 说明 | 默认值 |
|------|------|--------|
| --device-plugin-path | kubelet 设备插件目录路径 | /var/lib/kubelet/device-plugins |
| --device-list-strategy | 设备列表策略 | envvar |
| --device-id-strategy | 设备 ID 策略 | uuid |

### 配置文件

配置文件支持 YAML 格式，用于配置设备插件的行为和参数。

## 使用设备

### 整卡分配示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vastai-test-pod
spec:
  containers:
  - name: vastai-container
    image: vastai-test-image
    resources:
      limits:
        vastai.com/gpu: 1
```

### Time Slicing 分配示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vastai-timeslicing-pod
spec:
  containers:
  - name: vastai-container
    image: vastai-test-image
    resources:
      limits:
        vastai.com/gpu: 1
```

## 验证安装

1. 检查设备插件 Pod 状态
```bash
kubectl get pods -n vastai-plugin
```

2. 查看节点可分配资源
```bash
kubectl describe node {nodeName} | grep Allocatable -A 10
```

3. 检查设备健康状态
```bash
kubectl logs -n vastai-plugin vastai-device-plugin-daemonset-xxxxx
```

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 设备插件无法启动 | 检查 Docker/Containerd 运行时配置 |
| 设备无法识别 | 确认 PCIe 驱动已正确安装 |
| Pod 无法调度 | 检查节点标签和节点选择器配置 |
| 资源分配失败 | 验证 Time Slicing 配置是否正确 |

## 术语说明

- **Kubernetes (K8s)**: 容器编排平台
- **Pod**: K8s 中最小的部署单元
- **DaemonSet**: 确保所有节点运行一个 Pod 副本的控制器
- **Time Slicing**: 时间切片，允许多个任务共享同一设备资源
