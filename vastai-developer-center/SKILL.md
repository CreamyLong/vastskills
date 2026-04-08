---
name: "vastai-developer-center"
description: "瀚博半导体开发者中心部署指南。Invoke when user needs to deploy DeepSeek-V3/R1/V3.1 or Qwen3 series models on Vastai hardware with vLLM+VACC."
---

# 瀚博半导体开发者中心

## Overview

瀚博半导体开发者中心提供在 Vastai AI 加速器上部署大型语言模型的完整解决方案，支持 DeepSeek-V3/R1/V3.1 系列和 Qwen3 系列模型的高效推理。基于 vLLM+VACC 框架，提供一键部署、分步部署、性能测试和 WebUI 访问等功能。

## When to Use This Skill

This skill should be used when:
- 在 Vastai VA16/VA1L/VA10L 硬件上部署 DeepSeek 或 Qwen3 系列模型
- 使用 vLLM+VACC 框架进行模型推理服务部署
- 需要一键部署或分步部署模型服务
- 进行模型性能测试和精度验证
- 配置 Open WebUI 进行交互式访问
- 管理多模型、多实例的推理服务
- 优化模型推理性能（吞吐量、延迟）

## Core Capabilities

### 1. 硬件配置与要求

**硬件要求**:

| 模型规格 | 最低硬件配置要求 |
|----------|------------------|
| DeepSeek-V3/R1/V3.1 系列 | 单台 VA16（8*128G）服务器 |
| Qwen3 30B 系列（FP8） | 单卡 VA16 (128G)/单卡 VA1L (64G) / 单卡 VA10L (128G) |
| Qwen3 235B 系列 (FP8) | 单台 VA16（4*128G）服务器 |

**推荐操作系统**:

| 操作系统版本 | 对应内核版本 |
|--------------|--------------|
| Ubuntu 22.04 | `5.15.0-119-generic`、`5.15.0-139-generic` |
| UOS Server 20 | `4.19.90-2403.3.0.0270.87.uel20.x86_64` |
| KeyarchOS-5.8-SP2-U1 | `5.10.134-17.2.2.kos5.x86_64` |
| Kylin V10 | `4.19.90-89.11.v2401.ky10.aarch64` |

**推荐 CPU 型号**:

| 厂商 | CPU 型号 |
|------|----------|
| Intel | Xeon Platinum 8358, Xeon Gold 6330, Xeon Gold 6430, Xeon Gold 6530 |
| Hygon | C86 7375 32-core @3.0GHz, C86-4G (OPN:7470) 48-core @2.6GHz |
| AMD | EPYC 7543 32-Core, EPYC 7352 24-Core |
| Phytium | S5000C/64 @2.1GHz (dual CPU) * 2 |

**版本配套**:

| 组件 | 版本 |
|------|------|
| Driver | V3.3.0 |
| torch | 2.7.0+cpu |
| vllm | 0.9.2+cpu |
| vllm_vacc | VVI-25.11 |

### 2. 支持的模型列表

**DeepSeek 系列**:
- DeepSeek-V3
- DeepSeek-V3-0324
- DeepSeek-V3-Base
- DeepSeek-R1
- DeepSeek-R1-0528
- DeepSeek-V3.1
- DeepSeek-V3.1-Base
- DeepSeek-V3.1-Terminus

**Qwen3 系列**:
- Qwen3-30B-A3B-FP8
- Qwen3-30B-A3B-Instruct-2507-FP8
- Qwen3-30B-A3B-Thinking-2507-FP8
- Qwen3-Coder-30B-A3B-Instruct-FP8
- Tongyi-DeepResearch-30B-A3B-FP8
- Qwen3-235B-A22B-FP8
- Qwen3-235B-A22B-Instruct-2507-FP8
- Qwen3-235B-A22B-Thinking-2507-FP8

**模型限制**:
- DeepSeek-V3/R1/V3.1 系列：最大上下文长度 64K，最大输入长度 56K
- Qwen3 系列：
  - TP=2 时，最大上下文长度 64K
  - TP=4 或 16 时，最大上下文长度 128K
- 最大并发数：4

### 3. 模型下载

**安装 ModelScope**:

```bash
pip install modelscope -i https://mirrors.ustc.edu.cn/pypi/web/simple
export PATH=$PATH:~/.local/bin
```

**下载 DeepSeek 系列模型**:

```bash
modelscope download --model deepseek-ai/$Model_Name --local_dir $Path/$Model_Name
```

**下载 Qwen3 系列模型**（除 Tongyi-DeepResearch-30B-A3B-FP8 外）:

```bash
modelscope download --model Qwen/$Model_Name --local_dir $Path/$Model_Name
```

**下载 Tongyi-DeepResearch-30B-A3B-FP8**:

```bash
modelscope download --model iic/Tongyi-DeepResearch-30B-A3B --local_dir $Path/$Model_Name
```

**注意事项**:
- 模型文件较大，请确保磁盘空间充足
- 支持断点续传，异常中断后可重新执行继续下载
- Tongyi-DeepResearch-30B-A3B-FP8 启动前需经过 VACC 量化

### 4. 一键部署服务

**前提条件**:
- 已满足基础环境要求
- 已停止并删除 vllm_service 与 haproxy-server 同名容器（避免冲突）

**部署步骤**:

```bash
# 解压部署包
tar -xzvf vLLM_VACC_Guide-xxx.tar.gz
chmod +x vallmdeploy_xxx.run

# 一键部署
./vallmdeploy_xxx.run <MODEL_PARAM> <Model_Path>
```

**MODEL_PARAM 参数说明**:

| MODEL_PARAM | 说明 |
|-------------|------|
| DS3-V3 | DeepSeek-V3/V3.1 系列 |
| DS3-R1 | DeepSeek-R1 系列 |
| DS3-V3-MTP | DeepSeek-V3/V3.1 + MTP 技术 |
| DS3-R1-MTP | DeepSeek-R1 + MTP 技术 |
| Qwen3-TP2 | Qwen3-30B 系列，TP=2 |
| Qwen3-TP4 | Qwen3-30B 系列，TP=4 |
| Qwen3-TP16 | Qwen3-235B 系列，TP=16 |
| Qwen3-Instruct-2507-TP2 | Qwen3-30B-A3B-Instruct-2507-FP8, TP=2 |
| Qwen3-Instruct-2507-TP4 | Qwen3-30B-A3B-Instruct-2507-FP8, TP=4 |
| Qwen3-Instruct-2507-TP16 | Qwen3-235B-A22B-Instruct-2507-FP8, TP=16 |
| Qwen3-Thinking-2507-TP2 | Qwen3-30B-A3B-Thinking-2507-FP8, TP=2 |
| Qwen3-Thinking-2507-TP4 | Qwen3-30B-A3B-Thinking-2507-FP8, TP=4 |
| Qwen3-Thinking-2507-TP16 | Qwen3-235B-A22B-Thinking-2507-FP8, TP=16 |

### 5. 分步部署服务

**步骤 1: 安装 PCIe 驱动**

```bash
chmod +x ./vastai_driver_install_xxx.run
./vastai_driver_install_xxx.run install --setkoparam "dpm=1"
```

**步骤 2: 开启 DPM**

```bash
vasmi setconfig dpm=enable -d all
```

**步骤 3: 安装 vLLM_VACC**

```bash
tar -xzvf vLLM_VACC_Guide-xxx.tar.gz
chmod +x ./vllm_vacc_guide-xxx.bin
./vllm_vacc_guide-xxx.bin -i <path>
```

**步骤 4: 启动模型服务**

以 Qwen3-30B-A3B-Instruct-2507-FP8 为例：

```bash
cd /path/to_install/vllm_vacc_guide/service/haproxy/
python3 deploy.py --config ./config/Qwen3-Instruct-2507-TP4.yaml \
       --model /home/username/weights/Qwen3-30B-A3B-Instruct-2507-FP8
```

**deploy.py 参数说明**:

- `--config`: 读取内置配置文件启动服务
- `--model`: 原始模型权重路径
- `--instance`: 模型推理实例数（instance * tensor-parallel-size <= 推理核心数）
- `--tensor-parallel-size`: 张量并行数
  - DeepSeek V3/R1/V3.1: 仅支持 32
  - Qwen3 30B 系列：仅支持 2 或 4
  - Qwen3 235B 系列：仅支持 16
- `--port`: 模型服务端口
- `--management-port`: 管理端口
- `--max-batch-size-for-instance`: 每个实例的最大 Batch Size（最大 4）
- `--served-model-name`: 模型名称
- `--max-model-len`: 最大上下文长度
  - DeepSeek 系列：65536
  - Qwen3 系列：TP=2 时为 65536，TP=4 时为 131072
- `--reasoning-parser`: 推理解析器
- `--chat-template`: 聊天模板 Jinja 配置文件路径
- `--enable-benchmark`: 开启 Benchmark 测试
- `--benchmark-count`: Benchmark 测试运行次数（默认 5）
- `--dry-run`: 验证参数有效性
- `--disable-download-model`: 关闭自动下载模型
- `--enable-qwen3-rope-scaling`: 启用 ROPE 缩放（Qwen3 模型 max-model-len>32768 时必须启用）
- `--enable-auto-tool-choice`: 启用自动工具选择
- `--tool-call-parser`: 工具调用解析器
  - Qwen3 系列（除 Coder）: `hermes`
  - Qwen3-Coder-30B-A3B-Instruct-FP8: `qwen3_coder`
  - DeepSeek-R1-0528: `deepseek_v3` + tool_chat_template_deepseekr1.jinja
  - DeepSeek-V3-0324: `deepseek_v3` + tool_chat_template_deepseekv3.jinja
- `--enable-speculative-config`: 开启 MTP 模式（仅 DeepSeek-V3/V3.1/R1 系列）

**步骤 5: 查看日志**

```bash
tail -f /var/log/supervisor/vllm_serve_x.log
```

**步骤 6: 停止服务**

```bash
cd /path/to_install/vllm_vacc_guide/service/haproxy/
docker-compose -f docker-compose.yaml down
```

### 6. 性能测试

**vLLM 自带框架测试**:

```bash
docker exec -it vllm_service bash
cd /test/benchmark
export OPENAI_API_KEY="token-abc123"

python3 benchmark_serving.py \
    --host <IP> \
    --port 8000 \
    --model /weights/DeepSeek-V3-0324 \
    --dataset-name random \
    --num-prompts 5 \
    --random-input-len 128 \
    --ignore-eos \
    --random-output-len 1024 \
    --max-concurrency 1 \
    --served-model-name DeepSeek-V3 \
    --save-result \
    --result-dir ./benchmark_result \
    --result-filename result.json
```

**参数说明**:
- `--host`: vLLM 服务 IP 地址
- `--port`: vLLM 服务端口
- `--model`: 模型权重路径
- `--dataset-name`: 数据集名称
- `--num-prompts`: 测试输入数量
- `--random-input-len`: 输入序列长度
- `--ignore-eos`: 忽略 EOS Token
- `--random-output-len`: 输出序列长度
- `--max-concurrency`: 最大并发数
- `--served-model-name`: 模型名称
- `--save-result`: 保存测试结果
- `--result-dir`: 结果保存目录
- `--result-filename`: 结果文件名

**性能指标说明**:
- Maximum req: 最大并发数
- Duration: 请求测试耗时
- Successful req: 请求总数
- input tokens: 输入 Token 数量
- generated tokens: 输出 Token 数量
- Req throughput: 每秒处理请求数
- Output token throughput: 每秒输出 Token 数量
- Total Token throughput: 每秒生成 Token 数量
- Mean TTFT: 从请求到生成第一个 Token 的平均时间
- Mean TPOT: 生成每个输出 Token 的平均时间
- Decode Token throughput: Decode 阶段每秒输出 Token 数量
- Per-req Decoding token throughput: Decode 阶段平均每用户每秒输出 Token 数量

### 7. Open WebUI 配置

**步骤 1: 拉取镜像**

```bash
docker pull harbor.vastaitech.com/ai_deliver/vast-webui:latest
```

**步骤 2: 启动服务**

```bash
docker run -d \
    -v vast-webui:/app/backend/data \
    -e ENABLE_OLLAMA_API=False \
    --network=host \
    -e PORT=18080 \
    -e OPENAI_API_BASE_URL="http://127.0.0.1:8000/v1" \
    -e DEFAULT_MODELS="/weights/Qwen3-30B-A3B-FP8" \
    -e DEFAULT_LOCALE="cn" \
    --name vast-webui \
    --restart always \
    harbor.vastaitech.com/ai_deliver/vast-webui:latest
```

**参数说明**:
- `OPENAI_API_BASE_URL`: vLLM 服务地址
- `DEFAULT_MODELS`: 原始模型权重路径
- ARM 架构需使用 `harbor.vastaitech.com/ai_deliver/vast-webui:latest_arm`

**步骤 3: 访问页面**

通过 `http://HostIP:18080` 访问 Open WebUI，首次进入需设置管理员账号密码。

默认账号（如瀚博已提供环境）:
- 用户名：admin@vastai.com
- 密码：admin123

**步骤 4: 配置 vLLM 连接**

1. 在"管理员面板 > 设置 > 外部连接"页签单击"+"
2. 配置 vLLM 服务地址（格式：`http://HostIP:Port/v1`）
3. 输入密钥（任意值）
4. 设置模型地址（原始模型权重路径）
5. 保存配置

**步骤 5: 禁用自动功能**

在"管理员面板 > 设置 > 界面"页签禁用相关功能，防止 Open WebUI 自动调用大模型。

**步骤 6: 开启对话体验**

选择配置的模型进行对话测试。

## Usage Examples

### 示例 1: 一键部署 DeepSeek-R1 模型

```bash
# 下载模型
modelscope download --model deepseek-ai/DeepSeek-R1 --local_dir /data/models/DeepSeek-R1

# 一键部署
./vallmdeploy_xxx.run DS3-R1 /data/models/DeepSeek-R1
```

### 示例 2: 分步部署 Qwen3-30B 模型

```bash
# 安装驱动
./vastai_driver_install_xxx.run install --setkoparam "dpm=1"

# 开启 DPM
vasmi setconfig dpm=enable -d all

# 启动服务
cd /opt/vllm_vacc_guide/service/haproxy/
python3 deploy.py --config ./config/Qwen3-TP4.yaml \
       --model /data/models/Qwen3-30B-A3B-FP8 \
       --max-batch-size-for-instance 4 \
       --enable-benchmark
```

### 示例 3: 性能测试

```bash
# 进入容器
docker exec -it vllm_service bash

# 执行测试
cd /test/benchmark
export OPENAI_API_KEY="token-abc123"

python3 benchmark_serving.py \
    --host 192.168.1.100 \
    --port 8000 \
    --model /weights/Qwen3-30B-A3B-FP8 \
    --dataset-name random \
    --num-prompts 10 \
    --random-input-len 256 \
    --random-output-len 512 \
    --max-concurrency 4 \
    --served-model-name Qwen3-30B \
    --save-result \
    --result-dir ./results \
    --result-filename qwen3_perf.json
```

### 示例 4: 配置 Open WebUI

```bash
# 启动 Open WebUI
docker run -d \
    -v vast-webui:/app/backend/data \
    -e ENABLE_OLLAMA_API=False \
    --network=host \
    -e PORT=18080 \
    -e OPENAI_API_BASE_URL="http://192.168.1.100:8000/v1" \
    -e DEFAULT_MODELS="/weights/Qwen3-30B-A3B-FP8" \
    -e DEFAULT_LOCALE="cn" \
    --name vast-webui \
    harbor.vastaitech.com/ai_deliver/vast-webui:latest

# 访问 http://192.168.1.100:18080
```

## Best Practices

1. **部署前检查**:
   - 确认硬件配置满足要求
   - 检查磁盘空间是否充足
   - 验证网络和端口可用性
   - 停止可能冲突的容器

2. **性能优化**:
   - 根据硬件资源合理设置 tensor-parallel-size
   - 调整 max-batch-size-for-instance 优化吞吐量
   - 使用 MTP 技术提升 DeepSeek 系列性能
   - 启用 ROPE 缩放支持更长上下文

3. **资源管理**:
   - 使用 vasmi 监控设备状态
   - 合理分配推理核心数
   - 避免资源过度占用

4. **日志监控**:
   - 定期查看 vLLM 服务日志
   - 监控性能指标变化
   - 及时发现并处理异常

## Troubleshooting

**问题 1: 服务启动失败**

解决方案:
1. 检查 PCIe 驱动是否正确安装
2. 验证 DPM 是否已开启
3. 查看日志文件定位错误
4. 确认端口未被占用

**问题 2: 模型下载中断**

解决方案:
- 重新执行下载命令，支持断点续传
- 检查网络连接稳定性
- 确认磁盘空间充足

**问题 3: 性能不达标**

解决方案:
1. 检查 tensor-parallel-size 设置
2. 调整 batch size 参数
3. 验证硬件资源利用率
4. 使用 benchmark 工具定位瓶颈

**问题 4: Open WebUI 无法连接**

解决方案:
1. 检查 vLLM 服务是否正常运行
2. 验证 OPENAI_API_BASE_URL 配置
3. 确认网络和防火墙设置
4. 检查模型路径是否正确

## Related Documentation

- VAML API 参考手册
- Vasmi 工具使用指南
- Vastai Docker 安装指南
- Kubernetes 设备插件安装指南

## Glossary

- **vLLM**: 高性能大语言模型推理框架
- **VACC**: Vastai AI Compiler & Compute
- **TP (Tensor Parallel)**: 张量并行
- **MTP (Multi-Token Prediction)**: 多令牌预测
- **TTFT (Time To First Token)**: 首 Token 时间
- **TPOT (Time Per Output Token)**: 每输出 Token 时间
- **FP8**: 8 位浮点量化格式
- **DPM**: Dynamic Power Management
