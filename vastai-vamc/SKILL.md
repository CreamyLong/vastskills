---
name: "vastai-vamc"
description: "瀚博半导体 VAMC 3.4.3 模型转换工具使用指南。Invoke when user needs to convert deep learning models to Vastai format for deployment on Vastai hardware."
---

# 模型转换工具 VAMC 3.4.3 使用指南

VAMC (Vastai Model Converter) 是瀚博半导体提供的模型转换工具，用于将 PyTorch、ONNX 等格式的深度学习模型转换为适配瀚博硬件的离线模型格式（.vam）。

## 功能特性

- 支持 PyTorch、ONNX 等前端模型格式
- 支持 INT8/FP16 量化
- 支持动态 shape 编译
- 支持多卡分布式模型转换
- 提供丰富的预处理和后处理算子
- 支持自定义算子扩展

## 环境安装

### 环境准备

#### 开源框架要求

- Python: 3.8 或更高版本
- PyTorch: 1.10.0 或更高版本
- ONNX: 1.9.0 或更高版本

#### 操作系统要求

- Ubuntu 18.04/20.04/22.04
- CentOS 7/8
- Windows 10/11

#### 版本对应关系

| VAMC 版本 | PyTorch 版本 | ONNX 版本 | 驱动版本 |
|-----------|--------------|-----------|----------|
| 3.4.3     | 1.10.0-2.0.1 | 1.9.0+    | 2.8.0+   |

### Windows 环境下安装 VAMC

#### 1. 安装 VAMC 所需的 Windows 环境

- 安装 Visual Studio 2019 或更高版本
- 安装 Python 3.8+
- 安装 CUDA Toolkit（如使用 GPU）

#### 2. 安装 Vastai 软件

```powershell
# 运行 Vastai 安装程序
Vastai-Setup-x.x.x.exe

# 按照向导完成安装
```

#### 3. （可选）卸载 PCIe 驱动

```powershell
# 如需要重新安装驱动，先卸载旧版本
# 设备管理器 -> 显示适配器 -> 右键卸载设备
```

### Linux 环境下安装 VAMC

#### 1. 安装 LLVM

**Ubuntu 环境**:
```bash
# 安装 LLVM
sudo apt-get update
sudo apt-get install -y llvm-12 clang-12

# 设置环境变量
export LLVM_CONFIG=/usr/bin/llvm-config-12
```

**CentOS 环境**:
```bash
# 安装 LLVM
sudo yum install -y llvm clang

# 或使用源码编译安装
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-12.0.0/clang+llvm-12.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz
tar -xvf clang+llvm-12.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz
export PATH=$PWD/clang+llvm-12.0.0-x86_64-linux-gnu-ubuntu-20.04/bin:$PATH
```

#### 2. 安装 VastStream SDK

```bash
# 解压 SDK
tar -xzvf VastStream-SDK-x.x.x.tar.gz

# 安装
sudo ./install.sh

# 设置环境变量
source /opt/vastai/vaststream/setup.sh
```

#### 3. 安装 Torch 环境

```bash
# 安装 PyTorch
pip install torch==1.13.0 torchvision==0.14.0

# 验证安装
python -c "import torch; print(torch.__version__)"
```

#### 4. （可选）安装 torch_vacc 环境

```bash
# 安装瀚博 PyTorch 扩展
pip install torch_vacc-x.x.x-cp38-cp38-linux_x86_64.whl
```

#### 5. （可选）安装 PCIe 驱动

**dpkg 安装**:
```bash
sudo dpkg -i vastai-pcie-driver_xxx.deb
sudo modprobe vastai_pcie
```

**rpm 安装**:
```bash
sudo rpm -ivh vastai-pcie-driver-xxx.rpm
sudo modprobe vastai_pcie
```

**run 安装**:
```bash
chmod +x vastai-pcie-driver-xxx.run
sudo ./vastai-pcie-driver-xxx.run
```

#### 6. （可选）安装 Tools

```bash
# 安装瀚博工具包
sudo ./install_tools.sh
```

#### 7. 安装 VAMC

```bash
# 安装 VAMC
pip install vamc-3.4.3-py3-none-any.whl

# 验证安装
vamc --version
```

#### 8. 设置环境变量

```bash
# 添加到 ~/.bashrc
export VASTAI_HOME=/opt/vastai
export PATH=$VASTAI_HOME/vamc/bin:$PATH
export PYTHONPATH=$VASTAI_HOME/vamc/python:$PYTHONPATH

# 生效
source ~/.bashrc
```

## 模型转换配置文件说明

VAMC 使用 YAML 格式的配置文件来定义模型转换流程。

### 配置文件结构

```yaml
# 前端配置
frontend:
  checkpoint: "path/to/model.pth"  # 模型文件路径
  shape:                           # 输入形状
    input: [1, 3, 224, 224]
  type: "pytorch"                  # 模型类型: pytorch/onnx
  model_kwargs:                    # 模型加载参数
    tp: 1                          # 张量并行度
    b2s: false                     # batch to single
    split_embed: false             # 是否分割 embedding
    align_qkv: false               # 是否对齐 QKV
    onnx_cache: ""                 # ONNX 缓存路径
    model_arch: "resnet50"         # 模型架构
    from_onnx: false               # 是否从 ONNX 转换
    build_visual: false            # 是否构建可视化
    seq_lens: []                   # 序列长度列表
    dump_layers: []                # 要 dump 的层
    cached_length: 0               # 缓存长度
    insert_slice: false            # 是否插入 slice
  quantize:                        # 量化配置
    type: "int8"                   # 量化类型: int8/fp16/none
    device: "cuda:0"               # 校准设备
    calib_args:                    # 校准参数
      wbits: 8                     # 权重量化位数
      damp_percent: 0.01           # 阻尼百分比
      onegroup: false              # 是否使用单组
      blocksize: 128               # 块大小
      desc_act: false              # 是否降序激活
      true_sequential: true        # 是否真顺序
      model_file_base_name: ""     # 模型文件基础名
      model_file_kv_name: ""       # KV 缓存文件名
  dtype: "float32"                 # 数据类型

# 模型图配置
graph:
  extra_ops:                       # 额外算子
    - type: "custom_op"
      params:
        param1: value1

# 后端配置
backend:
  type: "vacc"                     # 后端类型
  merge_params: []                 # 合并参数
  merge_shapes: {}                 # 合并形状
  remove_params: []                # 移除参数
  dtype: "float32"                 # 输出数据类型
  device: "vacc"                   # 目标设备
  compile:                         # 编译配置
    gather_data_vccl_dsp_enable: false
    attention_split_num: 1
    ffn_split_num: 1
    data_type: "float32"
  quantize:                        # 后端量化配置
    calibrate_mode: "minmax"       # 校准模式
    quantize_per_channel: true     # 是否逐通道量化
    overflow_adaptive: true        # 是否自适应溢出
    calibrate_range: 0.9999        # 校准范围
    skip_conv_layers: []           # 跳过的卷积层
    exclude_layers: []             # 排除的层
    quantize_dsp_ops: true         # 是否量化 DSP 算子
  dynamic_compile:                 # 动态编译配置
    enable_graph_partition: false  # 是否启用图分割
    is_check_conv: false           # 是否检查卷积
    is_sr4k: false                 # 是否 4K 超分
    is_elic_gs: false              # 是否 ELIC GS

# 数据集配置
dataset:
  type: "image"                    # 数据集类型
  name: "imagenet"                 # 数据集名称
  path: "path/to/dataset"          # 数据集路径
  nsamples: 128                    # 校准样本数
  seqlen: 2048                     # 序列长度
  sampler:                         # 采样器配置
    suffix: [".jpg", ".png"]
    get_data_num: 1000
  process_ops: []                  # 预处理算子

# 工作空间配置
workspace:
  path: "./workspace"              # 工作目录
  env_recheck: false               # 是否重新检查环境
  pyenv: "python"                  # Python 环境
  workers: 4                       # 工作线程数

# 输出配置
name: "resnet50"                   # 输出模型名称
```

### 前端 frontend 字段参数说明

#### checkpoint

模型文件路径，支持 .pth、.pt、.onnx 等格式。

```yaml
frontend:
  checkpoint: "/path/to/model.pth"
```

#### shape

输入张量的形状定义。

```yaml
frontend:
  shape:
    input1: [1, 3, 224, 224]
    input2: [1, 3, 299, 299]
```

#### type

模型类型，可选值：
- `pytorch`: PyTorch 模型
- `onnx`: ONNX 模型

```yaml
frontend:
  type: "pytorch"
```

#### model_kwargs

模型加载时的额外参数。

| 参数 | 类型 | 说明 |
|------|------|------|
| tp | int | 张量并行度，用于大模型分布式推理 |
| b2s | bool | 是否将 batch 维度转为单样本处理 |
| split_embed | bool | 是否分割 embedding 层 |
| align_qkv | bool | 是否对齐 QKV 矩阵维度 |
| onnx_cache | str | ONNX 模型缓存路径 |
| model_arch | str | 模型架构名称 |
| from_onnx | bool | 是否从 ONNX 格式加载 |
| build_visual | bool | 是否生成可视化图 |
| seq_lens | list | 支持的序列长度列表 |
| dump_layers | list | 需要 dump 输出的层名 |
| cached_length | int | KV 缓存长度 |
| insert_slice | bool | 是否自动插入 slice 算子 |

```yaml
frontend:
  model_kwargs:
    tp: 2
    b2s: false
    model_arch: "llama2-7b"
    seq_lens: [512, 1024, 2048]
```

#### quantize

量化配置。

**type**: 量化类型
- `int8`: INT8 量化
- `fp16`: FP16 量化
- `none`: 不量化

**device**: 校准设备，如 `cuda:0`、`cpu`

**calib_args**: 校准参数
- `wbits`: 权重量化位数（4/8）
- `damp_percent`: GPTQ 阻尼系数
- `onegroup`: 是否使用单组量化
- `blocksize`: 量化块大小
- `desc_act`: 是否降序排列激活值
- `true_sequential`: 是否使用真顺序量化
- `model_file_base_name`: 量化模型保存名称
- `model_file_kv_name`: KV 缓存量化模型名称

```yaml
frontend:
  quantize:
    type: "int8"
    device: "cuda:0"
    calib_args:
      wbits: 8
      damp_percent: 0.01
      blocksize: 128
      desc_act: false
      true_sequential: true
```

#### dtype

模型数据类型：
- `float32`
- `float16`

```yaml
frontend:
  dtype: "float32"
```

### 模型图 graph 字段参数说明

#### extra_ops

添加额外的自定义算子。

```yaml
graph:
  extra_ops:
    - type: "custom_preprocess"
      params:
        scale: 255.0
        mean: [0.485, 0.456, 0.406]
```

### 后端 backend 字段参数说明

#### type

后端类型：
- `vacc`: 瀚博 VACC 后端

#### merge_params

合并参数列表，用于融合算子。

```yaml
backend:
  merge_params:
    - ["conv1", "bn1", "relu1"]
```

#### merge_shapes

合并形状定义。

```yaml
backend:
  merge_shapes:
    input: [1, 3, 224, 224]
```

#### remove_params

需要移除的参数列表。

```yaml
backend:
  remove_params:
    - "num_batches_tracked"
```

#### compile

编译配置参数：
- `gather_data_vccl_dsp_enable`: 是否启用 VCCL DSP 数据收集
- `attention_split_num`: Attention 头分割数
- `ffn_split_num`: FFN 分割数
- `data_type`: 编译数据类型

```yaml
backend:
  compile:
    attention_split_num: 4
    ffn_split_num: 4
    data_type: "float16"
```

#### quantize

后端量化配置：
- `calibrate_mode`: 校准模式（minmax/entropy/percentile）
- `quantize_per_channel`: 是否逐通道量化
- `overflow_adaptive`: 是否自适应处理溢出
- `calibrate_range`: 校准范围（0-1）
- `skip_conv_layers`: 跳过的卷积层名称列表
- `exclude_layers`: 排除量化的层名称列表
- `quantize_dsp_ops`: 是否量化 DSP 算子

```yaml
backend:
  quantize:
    calibrate_mode: "entropy"
    quantize_per_channel: true
    overflow_adaptive: true
    calibrate_range: 0.9999
    skip_conv_layers:
      - "conv1"
      - "layer1.0.conv1"
```

#### dynamic_compile

动态编译配置：
- `enable_graph_partition`: 是否启用图分割
- `is_check_conv`: 是否检查卷积算子
- `is_sr4k`: 是否为 4K 超分模型
- `is_elic_gs`: 是否为 ELIC GS 模型

### dataset 字段参数说明

#### type

数据集类型：
- `image`: 图像数据集
- `text`: 文本数据集
- `numpy`: NumPy 数组数据集

#### name

数据集名称，用于日志和输出。

#### path

数据集路径。

#### nsamples

校准使用的样本数量。

#### seqlen

序列长度（用于 NLP 模型）。

#### sampler

采样器配置：
- `suffix`: 文件后缀列表
- `get_data_num`: 获取数据数量

#### process_ops

预处理算子列表。

```yaml
dataset:
  type: "image"
  name: "imagenet"
  path: "/data/imagenet/calib"
  nsamples: 128
  sampler:
    suffix: [".jpg", ".jpeg", ".png"]
    get_data_num: 1000
  process_ops:
    - type: "resize"
      size: [256, 256]
    - type: "center_crop"
      size: [224, 224]
    - type: "normalize"
      mean: [0.485, 0.456, 0.406]
      std: [0.229, 0.224, 0.225]
```

### workspace 字段参数说明

#### path

工作目录，用于存放中间文件和日志。

#### env_recheck

是否重新检查环境。

#### pyenv

Python 环境路径。

#### workers

并行工作线程数。

```yaml
workspace:
  path: "./workspace"
  env_recheck: false
  pyenv: "python3"
  workers: 4
```

### name 字段参数说明

输出模型的名称。

```yaml
name: "resnet50_int8"
```

## 预处理算子说明

### DecodeImage

图像解码算子。

```yaml
- type: "decode_image"
  backend: "opencv"      # 解码后端: opencv/pil
  cv2_flags: 1           # OpenCV 读取标志
```

### Resize

图像缩放算子。

```yaml
- type: "resize"
  size: [256, 256]       # 目标尺寸 [width, height]
  adaptive_side: "short" # 自适应边: short/long
  interpolation: "bilinear"  # 插值方法: bilinear/bicubic/nearest
  backend: "opencv"
```

### CenterCrop

中心裁剪算子。

```yaml
- type: "center_crop"
  crop_size: [224, 224]  # 裁剪尺寸
  efficientnet_style: false  # 是否使用 EfficientNet 风格
  crop_padding: 32       # 裁剪边距
  interpolation: "bilinear"
  backend: "opencv"
```

### Pad

填充算子。

```yaml
- type: "pad"
  size: [224, 224]       # 目标尺寸
  size_divisor: 32       # 尺寸除数（用于检测模型）
  pad_to_square: false   # 是否填充为正方形
  pad_val: 0             # 填充值
```

### Normalize

归一化算子。

```yaml
- type: "normalize"
  mean: [0.485, 0.456, 0.406]  # 均值
  std: [0.229, 0.224, 0.225]   # 标准差
  div255: false            # 是否先除以 255
  norm_type: "mean_std"    # 归一化类型
```

### ToTensor

转换为张量算子。

```yaml
- type: "to_tensor"
```

### CvtColor

颜色空间转换算子。

```yaml
- type: "cvt_color"
  code: "bgr2rgb"        # 转换代码: bgr2rgb/rgb2bgr/bgr2gray
  backend: "opencv"
```

### ChannelSplit

通道分割算子。

```yaml
- type: "channel_split"
  index: 0               # 分割索引
```

### Clip

裁剪算子。

```yaml
- type: "clip"
  range: [0, 255]        # 裁剪范围
```

## 后处理算子说明

### softmax

Softmax 归一化。

```yaml
post_process:
  - type: "softmax"
    dim: -1              # 计算维度
```

### yolov5_nms

YOLOv5 非极大值抑制。

```yaml
post_process:
  - type: "yolov5_nms"
    num_classes: 80      # 类别数
    confidence_threshold: 0.25  # 置信度阈值
    nms_threshold: 0.45    # NMS 阈值
```

## 配置示例

### 固定单输入 Shape 配置示例

```yaml
# resnet50_static.yaml
frontend:
  checkpoint: "resnet50.pth"
  shape:
    input: [1, 3, 224, 224]
  type: "pytorch"
  dtype: "float32"

backend:
  type: "vacc"
  device: "vacc"
  dtype: "float32"

dataset:
  type: "image"
  name: "imagenet"
  path: "/data/imagenet/calib"
  nsamples: 128
  process_ops:
    - type: "resize"
      size: [256, 256]
    - type: "center_crop"
      crop_size: [224, 224]
    - type: "normalize"
      mean: [0.485, 0.456, 0.406]
      std: [0.229, 0.224, 0.225]

workspace:
  path: "./workspace"
  workers: 4

name: "resnet50_static"
```

### 动态单输入 Shape 配置示例

```yaml
# resnet50_dynamic.yaml
frontend:
  checkpoint: "resnet50.pth"
  shape:
    input:                          # 动态形状
      min: [1, 3, 224, 224]
      opt: [4, 3, 224, 224]
      max: [8, 3, 224, 224]
  type: "pytorch"
  dtype: "float32"

backend:
  type: "vacc"
  device: "vacc"
  dtype: "float32"
  dynamic_compile:
    enable_graph_partition: true

dataset:
  type: "image"
  name: "imagenet"
  path: "/data/imagenet/calib"
  nsamples: 128
  process_ops:
    - type: "resize"
      size: [256, 256]
    - type: "center_crop"
      crop_size: [224, 224]
    - type: "normalize"
      mean: [0.485, 0.456, 0.406]
      std: [0.229, 0.224, 0.225]

workspace:
  path: "./workspace"
  workers: 4

name: "resnet50_dynamic"
```

### INT8 量化配置示例

```yaml
# resnet50_int8.yaml
frontend:
  checkpoint: "resnet50.pth"
  shape:
    input: [1, 3, 224, 224]
  type: "pytorch"
  quantize:
    type: "int8"
    device: "cuda:0"
    calib_args:
      wbits: 8
      damp_percent: 0.01
      blocksize: 128
      desc_act: false
      true_sequential: true
  dtype: "float32"

backend:
  type: "vacc"
  device: "vacc"
  dtype: "int8"
  quantize:
    calibrate_mode: "entropy"
    quantize_per_channel: true
    overflow_adaptive: true
    calibrate_range: 0.9999

dataset:
  type: "image"
  name: "imagenet"
  path: "/data/imagenet/calib"
  nsamples: 256
  process_ops:
    - type: "resize"
      size: [256, 256]
    - type: "center_crop"
      crop_size: [224, 224]
    - type: "normalize"
      mean: [0.485, 0.456, 0.406]
      std: [0.229, 0.224, 0.225]

workspace:
  path: "./workspace"
  workers: 4

name: "resnet50_int8"
```

## 命令行使用

### 基本转换命令

```bash
# 使用配置文件转换
vamc convert --config resnet50.yaml

# 指定输出路径
vamc convert --config resnet50.yaml --output ./output/

# 启用详细日志
vamc convert --config resnet50.yaml --verbose
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `--config` | 配置文件路径 |
| `--output` | 输出目录 |
| `--verbose` | 详细输出 |
| `--debug` | 调试模式 |
| `--gpu` | 指定 GPU 设备 |
| `--workers` | 并行工作线程数 |

### 验证转换结果

```bash
# 验证模型
vamc verify --model resnet50.vam --input input.npy

# 对比精度
vamc benchmark --model resnet50.vam --dataset /data/test --metric accuracy
```

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 模型加载失败 | 检查模型路径和 PyTorch 版本兼容性 |
| 量化精度下降 | 增加校准样本数，调整校准模式 |
| 编译失败 | 检查 LLVM 安装和环境变量 |
| 内存不足 | 减少 batch size 或 workers 数量 |
| 动态 shape 不支持 | 检查模型是否包含动态控制流 |

## 相关文档

- VastStream SDK 用户手册
- VastStreamX 用户手册
- VAML API 参考手册
