---
name: "vastai-vamp"
description: "瀚博半导体 VAMP (Vastai Model Profiler) 模型性能分析和功耗测试工具。Invoke when user needs to profile model performance, test throughput, latency, power consumption, or accuracy on Vastai VACC hardware."
---

# VAMP 工具使用指南

VAMP（Vastai Model Profiler）是瀚博针对 VACC 硬件设备支持的模型推出的一款性能分析和功耗测试的工具。

## 功能特性

- 支持测试不同 Batch size 在单 Die 或多 Die 下的最大吞吐量
- 支持统计不同 Batch size 下模型推理时延和端到端的时延（平均、最大、最小）
- 支持统计指定分位数的模型推理时延和端到端时延
- 支持监控模型运行时硬件资源状态以及 AI 利用率
- 支持查询芯片显存占用情况
- 支持监测模型运行时瀚博加速卡的平均温度
- 支持监测模型运行时瀚博加速卡的平均功耗
- 支持在保证性能的情况下测试模型精度
- 支持预加载数据、指定输入数据列表
- 支持多进程运行模型

## 环境要求

### 前提条件

- 已获取 VAMP 工具包
- 已安装 V1.4.8 版本的 VastPipe

查看 VastPipe 版本：
```bash
ll /opt/vastai/vastpipe
```

### 操作系统支持

- Ubuntu 18.04/20.04 (x86 和 ARM)
- CentOS 7.2+ (x86 和 ARM)
- OpenEuler 22.03 (ARM)

## 安装步骤

1. 将 VAMP 工具包上传至服务器
```bash
# 上传 vamp 到 /home/username 目录
```

2. 增加可执行权限
```bash
chmod +x vamp
```

3. 更新环境变量
```bash
source /opt/vastai/vastpipe/vastpipe/bin/activate.sh
```

如无权限，先修改权限：
```bash
sudo chmod -R 774 /opt/vastai/vastpipe/vastpipe/bin/activate.sh
```

## 命令参考

### 基本命令格式

```bash
./vamp [options] ...
```

### 命令参数说明

| 参数 | 属性 | 说明 |
|------|------|------|
| -m, --model_prefix | 可选 | 设置模型三件套前缀名称，需补全相对路径或绝对路径。例如: /opt/vastai/vastpipe/data/models/resnet50/resnet50-rgb |
| --model_json | 可选 | 设置模型三件套 json 文件路径 |
| --model_param | 可选 | 设置模型三件套 param 文件路径 |
| --model_lib | 可选 | 设置模型三件套 lib 文件路径 |
| --hwconfig | 可选 | 设置 hwconfig 文件路径 |
| --vdsp_parameters | 必选 | 设置模型推理预处理文件路径 |
| -b, --batch_size | 可选 | 设置模型 Batch size 大小，默认值：1 |
| -i, --instance | 可选 | 设置每个 Die 下模型运行的实例个数或范围，默认值：1。例如：1:2 表示执行实例为 1 和 2 的模型推理 |
| -d, --device_id | 可选 | 设置 Die ID，默认值：0。可指定多个，如 [0,2,4] 或 1:3 |
| -c, --config | 可选 | 设置配置 VAMP 工具的 yaml 格式文件 |
| -g, --graph | 可选 | 高级功能，graph 文件（暂未支持） |
| -f, --file_report | 可选 | 设置保存运行结果的 yaml 文件路径 |
| -s, --shape | 可选 | 设置模型输入尺寸，格式为 [c,h,w] |
| --dtype | 可选 | 设置模型输入数据类型，默认 uint8。可选：int8, uint8, int16, uint16, int32, uint32, int64, uint64, float16, float32 |
| --iterations | 可选 | 设置模型推理次数，默认值：1024 |
| -p, --processes | 可选 | 设置模型推理进程数，默认值：1 |
| --verbose | 可选 | 是否打印日志信息，默认值：0 |
| --percentiles | 可选 | 设置统计时延的分位数，默认值：[50,90,95,99] |
| --datalist | 可选 | 设置输入数据列表，格式为 ny。指定后 --iterations 无效 |
| --cache_input | 可选 | 是否预加载数据至加速卡上，默认值：0 |
| --cache_output | 可选 | 是否缓存输出数据至加速卡上，默认值：0 |
| --path_output | 可选 | 设置数据输出文件路径，格式为 npz |

## 使用示例

### 示例 1：单 Die 下执行 VAMP

```bash
cd /home/username
./vamp -m /opt/vastai/vastpipe/data/models/resnet50/resnet50-torchvision-torchscript-int8-percentile-224-224-runstreamrgb \
       -s [3,224,224] -b 16 -i 3 \
       --vdsp_parameters ./data/vdsp_parameters/common.json
```

输出结果：
```
number of instances in each device: 3
devices: [0]
batch size: 16
ai utilize (%): 95.6022
temperature (°C): 44.6633
card power (W): 47.5994
die memory used (MB): 1166.85
throughput (qps): 3400.64
e2e latency (us): avg latency: 40977 min latency: 9272 max latency: 45738
model latency (us): avg latency: 26796 min latency: 9096 max latency: 31548
```

### 示例 2：多 Die 下执行 VAMP

```bash
./vamp -m /opt/vastai/vastpipe/data/models/resnet50/resnet50-torchvision-torchscript-int8-percentile-224-224-runstreamrgb \
       -s [3,224,224] -b 16 -i 3 \
       --vdsp_parameters ./data/vdsp_parameters/common.json \
       -d 0:3
```

### 示例 3：多 Die 多进程下执行 VAMP

```bash
./vamp -m /opt/vastai/vastpipe/data/models/resnet50/resnet50-rgb \
       -s [3,224,224] -b 16 -i 3 \
       --vdsp_parameters ./data/vdsp_parameters/common.json \
       -d 0:3 -p 4
```

### 示例 4：使用配置文件

创建 config.yaml：
```yaml
model_prefix: /opt/vastai/vastpipe/data/models/resnet50/resnet50-rgb
shape: [3,224,224]
batch_size: 16
instance: 3
vdsp_parameters: ./data/vdsp_parameters/common.json
device_id: 0:3
iterations: 1024
```

执行：
```bash
./vamp -c config.yaml
```

### 示例 5：测试模型精度

```bash
./vamp -m /opt/vastai/vastpipe/data/models/resnet50/resnet50-rgb \
       -s [3,224,224] -b 1 \
       --vdsp_parameters ./data/vdsp_parameters/common.json \
       --datalist ./data/input_list.ny \
       --path_output ./output/results.npz
```

## 输出指标说明

| 指标 | 说明 |
|------|------|
| ai utilize (%) | AI 利用率百分比 |
| temperature (°C) | 加速卡平均温度（摄氏度） |
| card power (W) | 加速卡平均功耗（瓦特） |
| die memory used (MB) | Die 显存占用（MB） |
| throughput (qps) | 吞吐量（每秒查询数） |
| e2e latency (us) | 端到端时延（微秒） |
| model latency (us) | 模型推理时延（微秒） |
| avg latency | 平均时延 |
| min latency | 最小时延 |
| max latency | 最大时延 |
| p50/p90/p95/p99 latency | 50%/90%/95%/99% 分位时延 |

## 注意事项

1. VAMP 工具命令需在 VAMP 工具所在目录下执行
2. 重新登录环境后需先更新环境变量再执行命令
3. 使用 --cache_input 时需判断数据量是否超过加速卡内存大小
4. 设置多个 Die ID 时，进程数需小于或等于 Die 的数量
5. 只设置一个 Die ID 时，进程数需小于或等于实例数量
6. Bert 相关模型必须配置 --dtype 为 int32

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 命令无法执行 | 检查环境变量是否已更新 |
| 模型加载失败 | 检查模型路径和文件是否存在 |
| 预处理文件错误 | 检查 vdsp_parameters 文件路径和格式 |
| 显存不足 | 减小 batch_size 或 instance 数量 |
| AI 利用率低 | 尝试增加进程数或多实例运行 |
