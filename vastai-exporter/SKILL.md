---
name: "vastai-exporter"
description: "瀚博半导体 Vastai Exporter 安装指南。Invoke when user needs to install and configure Vastai Exporter for Prometheus monitoring of Vastai AI accelerators."
---

# Vastai Exporter 安装指南

Vastai Exporter 是基于开源监控系统 Prometheus 开发的一个组件，为瀚博设备资源提供多种指标的实时监控，可用于获取设备内存、利用率、温度、频率以及设备在容器中的分配状况等信息。

## 简介

Vastai Exporter 作为 Prometheus 的一个 exporter，提供以下功能：

- 实时采集瀚博 AI 加速器的硬件指标
- 支持 Docker 和 Kubernetes 两种部署方式
- 提供设备内存、利用率、温度、频率等监控指标
- 支持容器化环境中的设备监控

## 安装前准备

### 环境要求

在安装 Vastai Exporter 之前，请确保满足以下要求：

- 已安装 PCIe 驱动 (版本 >= 0.1)
- 已安装 Vastai Docker (版本 >= 0.1)
- 已安装 Kubernetes (版本 >= 1.16)（仅 K8s 部署需要）

### 获取软件包

需要获取以下软件包：

- `vastai-exporter-{arch}-{version}.tar.gz`: Vastai Exporter 镜像包
- `vastai-exporter.yaml`: Vastai Exporter 配置文件

## 安装部署

### Docker 环境安装

#### 步骤 1: 加载镜像

```bash
# 加载镜像
docker load < vastai-exporter-{arch}-{version}.tar.gz

# 查看镜像 ID
docker images | grep vastai-exporter

# 打标签
docker tag {img_id} {local_dockerhub_url}/vastai/vastai-exporter:v1.0

# 推送到本地仓库
docker push {local_dockerhub_url}/vastai/vastai-exporter:v1.0
```

其中：
- `{img_id}`: Vastai Exporter 镜像 ID
- `{local_dockerhub_url}`: Docker Hub 的本地镜像仓库地址
- `vastai/vastai-exporter:v1.0`: 镜像的名称和标签

#### 步骤 2: 修改配置文件

修改 `vastai-exporter.yaml` 配置文件中的 containers 选项：

```yaml
containers:
  - image: {local_dockerhub_url}/vastai/vastai-exporter:v1.0
```

#### 步骤 3: 启动 Vastai Exporter

```bash
docker run -d --runtime=vastai -e VASTAI_VISIBLE_DEVICES=all --rm -p 9400:9400 \
  {local_dockerhub_url}/vastai/vastai-exporter:v1.0
```

参数说明：
- `-d`: 容器在后台运行
- `--runtime=vastai`: 指定 vastai 为容器低级运行时
- `--rm`: 容器停止后自动删除
- `-p 9400:9400`: 将容器的 9400 端口映射到宿主机
- `VASTAI_VISIBLE_DEVICES=all`: 可见所有设备

#### 步骤 4: 验证安装

```bash
curl localhost:9400/metrics
```

成功输出示例：

```
# HELP VASTAI_VA_DEV_TEMP max temperature (in C).
# TYPE VASTAI_VA_DEV_TEMP gauge
VASTAI_VA_DEV_TEMP{va="0-0",UUID="0-0",device="kchar:0",modelName="vastai VA1", container="",namespace="",pod=""} 55
```

### Kubernetes 环境安装

#### 步骤 1: 添加节点标签

```bash
kubectl label nodes {nodeName} vastai-device=vastai
```

#### 步骤 2: 创建 Deployment 和 Service

创建 `vastai-exporter.yaml` 文件：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: "vastai-exporter"
  namespace: "vastai-exporter"
  labels:
    app.kubernetes.io/name: "vastai-exporter"
    app.kubernetes.io/version: "1.0.0"
spec:
  updateStrategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app.kubernetes.io/name: "vastai-exporter"
      app.kubernetes.io/version: "1.0.0"
  template:
    metadata:
      labels:
        app.kubernetes.io/name: "vastai-exporter"
        app.kubernetes.io/version: "1.0.0"
      name: "vastai-exporter"
    spec:
      nodeSelector:
        vastai-device: vastai
      runtimeClassName: vastai
      containers:
      - name: vastai-exporter
        image: {local_dockerhub_url}/vastai/vastai-exporter:v1.0
        ports:
        - containerPort: 9400
          name: http
        env:
        - name: VASTAI_VISIBLE_DEVICES
          value: "all"
        resources:
          limits:
            vastai.com/va: 1
          requests:
            vastai.com/va: 1
---
apiVersion: v1
kind: Service
metadata:
  name: vastai-exporter
  namespace: vastai-exporter
  labels:
    app.kubernetes.io/name: vastai-exporter
spec:
  selector:
    app.kubernetes.io/name: vastai-exporter
  ports:
  - name: http
    port: 9400
    targetPort: http
  clusterIP: None
```

#### 步骤 3: 应用配置

```bash
kubectl apply -f vastai-exporter.yaml
```

#### 步骤 4: 验证部署

```bash
# 查看 Pod 状态
kubectl get pods -n vastai-exporter

# 查看 Service
kubectl get svc -n vastai-exporter
```

## 修改采集频率

通过设置环境变量 `COLLECT_INTERVAL` 来修改采集频率（单位：秒）：

```bash
docker run -d --runtime=vastai -e VASTAI_VISIBLE_DEVICES=all \
  -e COLLECT_INTERVAL=30 \
  -p 9400:9400 \
  {local_dockerhub_url}/vastai/vastai-exporter:v1.0
```

默认采集频率为 60 秒。

## 获取监控指标

### 端口转发方式

```bash
# Kubernetes 环境
kubectl port-forward svc/vastai-exporter 9400:9400 -n vastai-exporter

# 访问指标
curl localhost:9400/metrics
```

### 使用 Prometheus 插件

配置 Prometheus 的 `prometheus.yml`：

```yaml
scrape_configs:
  - job_name: 'vastai-exporter'
    static_configs:
      - targets: ['<node-ip>:9400']
    scrape_interval: 30s
```

## 参数配置说明

### 环境变量参数

| 参数名 | 说明 | 默认值 |
|--------|------|--------|
| VASTAI_VISIBLE_DEVICES | 可见设备列表 | all |
| COLLECT_INTERVAL | 采集频率（秒） | 60 |
| LOG_LEVEL | 日志级别 | info |

### 数据监控指标

主要监控指标包括：

- `VASTAI_VA_DEV_TEMP`: 设备温度
- `VASTAI_VA_DEV_POWER`: 设备功耗
- `VASTAI_VA_DEV_UTILIZATION`: 设备利用率
- `VASTAI_VA_DEV_MEMORY_USED`: 内存使用量
- `VASTAI_VA_DEV_MEMORY_TOTAL`: 总内存
- `VASTAI_VA_DEV_CLOCK`: 设备时钟频率

## FAQ

### Runtime 版本不一致

**问题**: 容器启动失败，提示 runtime 版本不一致。

**解决方案**:
1. 检查 PCIe 驱动版本
2. 重新安装 Vastai Docker
3. 确保 Docker 配置正确

```bash
# 检查 Docker runtime 配置
cat /etc/docker/daemon.json

# 重启 Docker 服务
sudo systemctl restart docker
```

## 术语/缩略语

- **Prometheus**: 开源的系统监控和告警工具
- **Kubernetes**: 容器编排平台
- **Pod**: Kubernetes 中最小的部署单元
- **SoC**: System on Chip
- **SMCU**: System Management Control Unit

## 相关文档

- Kubernetes 设备插件安装指南
- Vastai Docker 安装指南
- VAML API 参考手册
