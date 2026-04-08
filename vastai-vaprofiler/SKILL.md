---
name: "vastai-vaprofiler"
description: "瀚博半导体 Vaprofiler 芯片监测与分析工具。Invoke when user needs to monitor and analyze Video encode/decode throughput, AI utilization, and system performance on Vastai accelerators."
---

# Vaprofiler 工具使用指南

Vaprofiler 是芯片监测与分析工具，用于查看系统中主机、AI 和 Video 的吞吐量和利用率等信息。

## 功能特性

- Video 编码相关信息监控
- Video 解码相关信息监控
- 资源利用率相关信息监控
- 查询设备信息
- 查询编解码参数
- 支持热插拔模式

## 运行环境

### 硬件环境

- CPU: Intel i5-4570（或更高配置），或性能相当的 AMD/ARM 架构 CPU
- Memory: 8 GB 或以上

### 支持的卡类型

**SV100 系列**:
- VA1、VA1A、VA1L、VA1U、VA1V、VA10、VA10L、VE1S、VA1M、VA12、VA16、VA10S

**SGPU100 系列**:
- VG1000、DC1000、VG1000c、DC1000c

### 软件环境

- Ubuntu 18.04/20.04 (x86 和 ARM)
- CentOS 7.2+ (x86 和 ARM)
- OpenEuler 22.03 (ARM)
- NDK
- Win64

## 安装步骤

### 自动安装

一般情况下，业务软件安装完成后，工具的依赖库和工具自动安装。

### 手动安装

1. 解压安装包
```bash
tar -xzvf tools-xxx-fusion-tools.tar.gz
```

2. 安装 Tools
```bash
cd tools-xxx-fusion/tools
./Tools-xxx-sv100.bin
```

显示 "Tools install succeeded." 说明安装成功。

3. 设置环境变量
```bash
source /etc/profile
```

注意：当前设置的环境变量仅在当前会话有效。以下情况需重新执行：
- 系统重启
- 用户关闭终端窗口
- 用户重新登录

4. 验证版本
```bash
vaprofiler --version
```

## 卸载

切换至安装目录，执行卸载脚本：
```bash
cd /opt/va/vaststream
./uninstall.tools.sh
```

## 命令参考

### 基本命令格式

```bash
vaprofiler <command> [<args>]
```

### 全局选项

| 选项 | 说明 |
|------|------|
| -d <AIC0,AIC1[,..]> | 指定 AIC 设备 |
| -i <Die0,Die1[,..]> | 指定 Die |
| --hotplug | 启用热插拔兼容模式 |
| --loop <count,time_interval(ms)> | 指定执行次数和时间间隔 |
| --version | 打印版本信息 |
| --help | 打印帮助信息 |

### 子命令列表

| 命令 | 说明 |
|------|------|
| list | 查询主机上所有设备 |
| fmon | 监控性能 |
| video | 监控视频参数 |
| vgpu | 监控 vgpu |

## 通用命令

### 查询帮助

```bash
vaprofiler --help
```

输出示例：
```
version: vaprofiler 0.0.0
Usage: vaprofiler <command> [<args>]
[--version] Print version information
[-help] Print help information
[-d <AIC0,AIC1[,..]>] Specify AIC
[-i <Die0,Die1[,..]>] Specify Die
[-hotplug] Enable hot-plug
[--loop <count,time_interval(ms)>] Specify execution counts

These are common vaprofiler commands used in various situations:
  list    Query all devices on the host
  fmon    Monitor performance
  video   Monitor video params
  vgpu    Monitor vgpu
```

### 查询版本

```bash
vaprofiler --version
```

## 常用功能

### 查询设备信息

```bash
vaprofiler list
```

输出示例：
```
Device List:
+--------+--------+--------+--------+--------+--------+--------+--------+
| Dev ID | Name   | Die Num| Status | Temp   | Power  | Memory | Util   |
+--------+--------+--------+--------+--------+--------+--------+--------+
| 0      | VA1    | 4      | OK     | 45°C   | 50W    | 8GB    | 85%    |
| 1      | VA1    | 4      | OK     | 43°C   | 48W    | 8GB    | 80%    |
+--------+--------+--------+--------+--------+--------+--------+--------+
```

### 查询编解码参数

#### Video 1 监控

```bash
vaprofiler video 1
```

输出示例：
```
Video Encode/Decode Monitor:
+--------+--------+--------+--------+--------+--------+--------+--------+
| Die ID | Type   | Codec  | Resolution | FPS    | Bitrate| Status | Error  |
+--------+--------+--------+--------+--------+--------+--------+--------+
| 0      | Encode | H264   | 1920x1080  | 60     | 4Mbps  | Running| 0      |
| 1      | Decode | H265   | 3840x2160  | 30     | 8Mbps  | Running| 0      |
+--------+--------+--------+--------+--------+--------+--------+--------+
```

#### Video 2 监控

```bash
vaprofiler video 2
```

输出示例：
```
Video Detailed Statistics:
+--------+--------+--------+--------+--------+--------+--------+--------+
| Die ID | Session| Input  | Output | Dropped| Queue  | Latency| Throughput|
+--------+--------+--------+--------+--------+--------+--------+--------+
| 0      | 1      | 1000   | 998    | 2      | 5      | 20ms   | 60fps   |
| 1      | 2      | 500    | 500    | 0      | 2      | 15ms   | 30fps   |
+--------+--------+--------+--------+--------+--------+--------+--------+
```

### 查询资源利用率

```bash
vaprofiler fmon
```

输出示例：
```
Performance Monitor:
+--------+--------+--------+--------+--------+--------+--------+--------+
| Die ID | AI Util| VDSP   | VEMCU  | VDMCU  | Memory | PCIe BW| DDR BW |
+--------+--------+--------+--------+--------+--------+--------+--------+
| 0      | 85.5%  | 80.2%  | 75.3%  | 70.1%  | 60.5%  | 2.5GB/s| 8.2GB/s|
| 1      | 82.3%  | 78.5%  | 72.1%  | 68.9%  | 58.2%  | 2.3GB/s| 7.8GB/s|
+--------+--------+--------+--------+--------+--------+--------+--------+
```

带循环监控：
```bash
# 每 1000ms 执行一次，共执行 100 次
vaprofiler fmon --loop 100,1000
```

### VGPU 监控

```bash
vaprofiler vgpu
```

输出示例：
```
VGPU Monitor:
+--------+--------+--------+--------+--------+--------+--------+--------+
| Die ID | Context| State  | Memory | Command| Shader | Texture| Render |
+--------+--------+--------+--------+--------+--------+--------+--------+
| 0      | 1      | Active | 512MB  | 1000   | 85%    | 60%    | 70%    |
| 1      | 2      | Idle   | 256MB  | 500    | 0%     | 0%     | 0%     |
+--------+--------+--------+--------+--------+--------+--------+--------+
```

## 使用示例

### 示例 1：查询所有设备

```bash
vaprofiler list
```

### 示例 2：监控指定设备的性能

```bash
vaprofiler fmon -d 0
```

### 示例 3：监控指定 Die 的视频参数

```bash
vaprofiler video 1 -i 0,1
```

### 示例 4：热插拔场景下监控

```bash
vaprofiler fmon -d 0 --hotplug
```

注意：热插拔模式仅适用于 VA1M 卡。

### 示例 5：循环监控性能

```bash
# 每 2 秒执行一次，持续监控
vaprofiler fmon --loop 0,2000
```

### 示例 6：监控多个设备

```bash
vaprofiler list -d 0,1,2
```

## 输出指标说明

### 设备信息指标

| 指标 | 说明 |
|------|------|
| Dev ID | 设备 ID |
| Name | 设备名称 |
| Die Num | Die 数量 |
| Status | 设备状态 |
| Temp | 设备温度 |
| Power | 设备功耗 |
| Memory | 内存使用情况 |
| Util | 利用率 |

### 视频编解码指标

| 指标 | 说明 |
|------|------|
| Type | 类型（Encode/Decode） |
| Codec | 编码格式（H264/H265/VP9/AV1） |
| Resolution | 分辨率 |
| FPS | 帧率 |
| Bitrate | 码率 |
| Status | 运行状态 |
| Error | 错误计数 |

### 性能监控指标

| 指标 | 说明 |
|------|------|
| AI Util | AI 利用率 |
| VDSP | VDSP 利用率 |
| VEMCU | VEMCU 利用率 |
| VDMCU | VDMCU 利用率 |
| Memory | 内存利用率 |
| PCIe BW | PCIe 带宽 |
| DDR BW | DDR 带宽 |

## 注意事项

1. 热插拔模式仅适用于 VA1M 卡
2. 设置环境变量后仅在当前会话有效
3. 系统重启或重新登录后需重新设置环境变量
4. 使用循环监控时注意设置合适的时间间隔
5. 监控大量设备时可能占用较多系统资源

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 无法识别设备 | 检查驱动是否正确安装 |
| 监控数据为 0 | 确认设备是否有任务在运行 |
| 热插拔失败 | 确认是否为 VA1M 卡 |
| 命令无法执行 | 检查环境变量是否已设置 |
| 输出显示异常 | 尝试更新到最新版本 |
