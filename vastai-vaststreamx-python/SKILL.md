---
name: "vastai-vaststreamx-python"
description: "瀚博半导体 VastStreamX 2.8.16 Python 用户手册。Invoke when user needs to develop deep learning inference applications using VastStreamX Python API on Vastai hardware."
---

# VastStreamX 2.8.16 用户手册 (Python)

VastStreamX 是 VastStream SDK 的更高层次封装，提供了 Stream 管理、内存管理、模型加载与执行、算子加载与执行、多媒体数据处理等 Python API，用于实现目标识别、图像分类、视频增强等功能。

## 功能特性

- Stream 管理：同步/异步推理流程管理
- 内存管理：Device/Host 内存分配与管理
- 模型加载与执行：支持静态和动态模型
- 算子加载与执行：内置算子和自定义算子支持
- 多媒体数据处理：视频编解码、图像处理
- 自动资源释放：对象销毁时自动释放资源
- 支持异步、可重入同步操作

## 核心类说明

### Image 类

用于存储和管理 Device 侧的图像帧数据。

```python
import vaststreamx as vsx

# 创建 Image 对象
image = vsx.Image()

# 从数据创建
image.adopt_data(data, width, height, format=vsx.ImageFormat.BGR)

# 从其他 Image 复制
image.copy_from(src_image)

# 克隆
cloned = image.clone()

# 获取属性
print(f"Format: {image.format()}")
print(f"ColorSpace: {image.color_space()}")
print(f"Height: {image.height()}")
print(f"Width: {image.width()}")
```

### Tensor 类

用于存储和管理 Device 侧的张量数据。

```python
import numpy as np

# 创建 Tensor
tensor = vsx.Tensor(context, shape=(1, 3, 224, 224), dtype=vsx.DataType.FLOAT32)

# 从 numpy 创建
np_array = np.random.randn(1, 3, 224, 224).astype(np.float32)
tensor = vsx.Tensor.from_numpy(context, np_array)

# 转换为 numpy
result = tensor.to_numpy()

# 获取形状
shape = tensor.shape()
print(f"Shape: {shape}")

# 重塑形状
tensor.reshape((1, 3, 112, 112))
```

### Model 类

用于加载和管理推理模型。

```python
# 创建静态模型
model = vsx.Model(context, "/path/to/resnet50.model")

# 创建动态模型
model = vsx.Model(context, "/path/to/yolo.model", max_batch_size=8)

# 设置 batch size（动态模型）
model.set_batch_size(4)

# 获取模型信息
print(f"Input count: {model.input_count()}")
print(f"Output count: {model.output_count()}")
print(f"Input shape: {model.input_shape_by_index(0)}")
print(f"Is dynamic: {model.is_dynamic()}")
```

### Graph 类

用于构建推理流程的计算图。

```python
# 创建 Graph
graph = vsx.Graph()

# 添加输入算子
graph.add_input_operator(input_op)

# 添加算子
graph.add_operator(op1)
graph.add_operator(op2)

# 批量添加算子
graph.add_operators([op1, op2, op3])
```

### Stream 类

用于管理推理流程的执行。

```python
# 创建 Stream
stream = vsx.Stream(context)

# 注册非模型算子输出
stream.register_none_model_operator_output(op, output_tensor)

# 注册模型算子输出
stream.register_model_operator_output(model_op, output_tensors)

# 构建推理流程
stream.build(graph)

# 同步执行
outputs = stream.run_sync(inputs)

# 异步执行
stream.run_async(inputs)
stream.wait_until_done()
outputs = stream.get_operator_output(model_op)
```

### Operator 类

#### BuildInOperator（内置算子）

```python
# 创建内置算子
resize_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_RESIZE)

# 设置属性
resize_op.set_attribute(vsx.AttrKey.RESIZE_HEIGHT, 224)
resize_op.set_attribute(vsx.AttrKey.RESIZE_WIDTH, 224)

# 常用内置算子
cvt_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_CVT_COLOR)
norm_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_NORM_TENSOR)
crop_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_CROP)
flip_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_FLIP)
```

**融合算子**:
```python
# YUV NV12 融合算子
fusion_op = vsx.BuildInOperator(
    vsx.BuildInOperatorType.FUSION_OP_YUV_NV12_RESIZE_CVTCOLOR_CROP_NORM_TENSOR
)

# RGB 融合算子
fusion_op = vsx.BuildInOperator(
    vsx.BuildInOperatorType.FUSION_OP_RGB_RESIZE_CVTCOLOR_NORM_TENSOR
)
```

#### ModelOperator（模型算子）

```python
# 从 Model 创建模型算子
model_op = vsx.ModelOperator(model)
```

#### CustomOperator（自定义算子）

```python
# 创建自定义算子
custom_op = vsx.CustomOperator("MyCustomOp")

# 设置回调函数
def callback(input_data, output_data):
    # 处理数据
    pass

custom_op.set_callback(callback)

# 设置配置
custom_op.set_config(config_dict)
```

## 环境安装

### Linux 环境安装

#### 1. 安装 PCIe 驱动

**Ubuntu**:
```bash
# 安装驱动包
sudo dpkg -i vastai-pcie-driver_xxx.deb

# 加载驱动
sudo modprobe vastai_pcie

# 查看版本
cat /sys/module/vastai_pcie/version
```

**CentOS**:
```bash
# 安装驱动包
sudo rpm -ivh vastai-pcie-driver-xxx.rpm

# 加载驱动
sudo modprobe vastai_pcie
```

#### 2. 安装 VastStream SDK

```bash
# 解压
tar -xzvf VastStream-SDK-xxx.tar.gz

# 安装
sudo ./install.sh

# 设置环境变量
source /opt/vastai/vaststream/setup.sh
```

#### 3. 安装 VastStreamX

```bash
# 解压
tar -xzvf VastStreamX-xxx.tar.gz

# 安装
sudo ./install.sh

# 设置环境变量
source /opt/vastai/vaststreamx/setup.sh
```

## 快速入门：图像分类

### 开发流程

```python
import vaststreamx as vsx
import numpy as np

# 1. 初始化系统
vsx.init_system()

# 2. 创建设备上下文
context = vsx.Context.create(0)  # 使用设备 0

# 3. 加载模型
model = vsx.Model(context, "/path/to/resnet50.model")

# 4. 创建模型算子
model_op = vsx.ModelOperator(model)

# 5. 创建预处理算子
resize_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_RESIZE)
resize_op.set_attribute(vsx.AttrKey.RESIZE_HEIGHT, 224)
resize_op.set_attribute(vsx.AttrKey.RESIZE_WIDTH, 224)

cvt_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_CVT_COLOR)
cvt_op.set_attribute(vsx.AttrKey.COLOR_CVT_CODE, vsx.ColorCvtCode.COLOR_BGR2RGB)

norm_op = vsx.BuildInOperator(vsx.BuildInOperatorType.SINGLE_OP_NORM_TENSOR)
norm_op.set_attribute(vsx.AttrKey.NORM_TYPE, vsx.NormType.NORM_MEAN_STD)
norm_op.set_attribute(vsx.AttrKey.NORM_MEAN, [0.485, 0.456, 0.406])
norm_op.set_attribute(vsx.AttrKey.NORM_STD, [0.229, 0.224, 0.225])

# 6. 构建计算图
graph = vsx.Graph()
graph.add_input_operator(resize_op)
graph.add_operator(cvt_op)
graph.add_operator(norm_op)
graph.add_operator(model_op)

# 7. 创建 Stream 并构建
stream = vsx.Stream(context)
stream.build(graph)

# 8. 准备输入数据
image_data = np.random.randint(0, 256, (224, 224, 3), dtype=np.uint8)
input_image = vsx.Image()
input_image.adopt_data(image_data, 224, 224, vsx.ImageFormat.BGR)

inputs = [input_image]

# 9. 执行推理
outputs = stream.run_sync(inputs)

# 10. 处理输出结果
result = outputs[0].to_numpy()
print(f"Output shape: {result.shape}")

# 11. 释放资源（自动）
```

## 同步推理 vs 异步推理

### 同步推理

```python
stream = vsx.Stream(context)
stream.build(graph)

# 同步执行，阻塞直到完成
outputs = stream.run_sync(inputs)

# 直接使用输出结果
process_output(outputs)
```

### 异步推理

```python
stream = vsx.Stream(context)
stream.build(graph)

# 异步执行，立即返回
stream.run_async(inputs)

# 执行其他任务...

# 等待完成
stream.wait_until_done()

# 获取输出
outputs = stream.get_operator_output(model_op)
process_output(outputs)
```

## 视频处理

### VideoCapture（视频捕获）

```python
# 打开视频文件
capture = vsx.VideoCapture("/path/to/video.mp4")

# 检查是否成功打开
if not capture.is_opened():
    print("Failed to open video")
    
# 读取帧
while True:
    ret, frame = capture.read()
    if not ret:
        break
    # 处理帧
    process_frame(frame)

# 释放
capture.release()
```

### VideoDecoder（视频解码）

```python
# 创建解码器
decoder = vsx.VideoDecoder(codec_type=vsx.CodecType.H264)

# 发送编码数据
decoder.send_data(packet_data)

# 接收解码后的图像
while True:
    image = decoder.recv_image(timeout_ms=1000)
    if image is None:
        break
    process_image(image)

# 停止发送
decoder.stop_send_data()
```

### VideoEncoder（视频编码）

```python
# 配置编码参数
config = vsx.VideoEncoderAdvancedConfig()
config.codec_type = vsx.CodecType.H264
config.width = 1920
config.height = 1080
config.fps = 30
config.bitrate = 4000000

# 创建编码器
encoder = vsx.VideoEncoder(config)

# 发送图像
encoder.send_image(image)

# 接收编码数据
while True:
    data = encoder.recv_data(timeout_ms=1000)
    if data is None:
        break
    process_encoded_data(data)

# 停止发送
encoder.stop_send_image()
```

## 硬件信息查询

### Card 类

```python
# 获取所有卡
cards = vsx.get_all_cards()

for card in cards:
    # 获取 UUID
    uuid = card.get_uuid()
    print(f"Card UUID: {uuid}")
    
    # 获取卡类型
    card_type = card.get_card_type()
    
    # 获取 PCIe 信息
    pci_info = card.get_pci_info()
    
    # 获取功耗
    power = card.get_card_power()
    
    # 获取风扇速度
    fan_speed = card.get_fan_speed_level()
    
    # 获取 Die 数量
    die_count = card.get_die_count()
```

### Die 类

```python
# 获取 Die
die = card.get_die_by_index(0)

# 获取时钟频率
freq = die.get_clock_frequency()
print(f"Clock: {freq}")

# 获取温度
temp = die.get_temperature()
print(f"Temperature: {temp}°C")

# 获取功耗
power = die.get_power()
print(f"Power: {power / 1000:.2f} W")

# 获取内存使用
memory = die.get_memory()
print(f"Memory: {memory.used}/{memory.total} MB")

# 获取利用率
util = die.get_utilization()
print(f"Utilization: {util}%")

# 获取运行中的进程
procs = die.get_running_processes()
```

## 内存管理

### Device 内存分配

```python
# 分配 Device 内存
dev_ptr = vsx.malloc_device(context, size)

# 分配扩展 Device 内存
dev_ptr_ext = vsx.malloc_device_ext(context, size, flags)

# 释放 Device 内存
vsx.free_device(context, dev_ptr)
vsx.free_device_ext(context, dev_ptr_ext)
```

### Host 内存分配

```python
# 分配 Host 内存
host_ptr = vsx.malloc_host(context, size)

# 释放 Host 内存
vsx.free_host(context, host_ptr)
```

### 内存拷贝

```python
# Host to Device
vsx.memcpy(context, dev_ptr, host_ptr, size, vsx.MemcpyKind.HOST_TO_DEVICE)

# Device to Host
vsx.memcpy(context, host_ptr, dev_ptr, size, vsx.MemcpyKind.DEVICE_TO_HOST)

# Device to Device
vsx.memcpy_device(context, dst_ptr, src_ptr, size)

# 设置内存
vsx.memset_device(context, dev_ptr, value, size)
```

## 常用数据结构

### StreamBalanceMode

```python
class StreamBalanceMode:
    BALANCE_MODE_NONE = 0         # 不启用负载均衡
    BALANCE_MODE_ROUND_ROBIN      # 轮询模式
    BALANCE_MODE_LEAST_LOAD       # 最小负载模式
```

### BuildInOperatorType

常用算子类型：
- 单算子：`SINGLE_OP_*`
- 融合算子：`FUSION_OP_*`

### ImageFormat

```python
class ImageFormat:
    BGR = 0
    RGB
    NV12
    NV21
    YUV420P
    GRAY
```

### DataType

```python
class DataType:
    INT8 = 0
    UINT8
    INT16
    UINT16
    INT32
    UINT32
    FLOAT16
    FLOAT32
```

## 注意事项

1. **资源管理**: VastStreamX 会自动管理资源，对象销毁时会自动释放资源
2. **线程安全**: Stream 对象不是线程安全的，每个线程应使用独立的 Stream
3. **内存对齐**: Device 内存分配会自动对齐
4. **错误处理**: 使用 try-except 捕获异常进行错误处理
5. **性能优化**: 
   - 使用融合算子减少数据传输
   - 合理设置 batch size
   - 使用异步执行提高吞吐量

## 相关文档

- VastStreamX C++ 用户手册
- VAML API 参考手册
- VAMP 工具使用指南
