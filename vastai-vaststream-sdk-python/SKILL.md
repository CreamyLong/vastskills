---
name: "vastai-vaststream-sdk-python"
description: "瀚博半导体 VastStream SDK 2.0 Python 用户手册。Invoke when user needs to develop deep learning inference and video codec applications using VastStream SDK Python API on Vastai hardware."
---

# VastStream SDK 2.0 用户手册 (Python)

VastStream SDK 是一套用于在瀚博设备平台上开发深度神经网络推理及视频编解码应用的 Python API 库，提供运行资源管理、内存管理、模型加载与执行、算子加载与执行、媒体数据处理等功能。

## 软件栈架构

VastStream SDK Python 版本包含以下核心模块：

- **VACM**: 运行资源管理（设备上下文、数据容器、内存管理）
- **VACE**: 图像运算操作（算子库）
- **VACL**: 推理应用开发（模型加载、Graph、Stream）
- **VAME**: 媒体数据处理（视频编解码）
- **VAML**: 设备资源管理（硬件信息查询）

## 基本概念

### Host 与 Device

- **Host**: 主机端，指 CPU 和系统内存
- **Device**: 设备端，指瀚博 AI 加速器

### 同步/异步

- **同步**: 调用后立即执行，阻塞直到完成
- **异步**: 调用后立即返回，通过回调或等待机制获取结果

### Context

设备上下文，管理设备资源和内存。一个 Device 可以有多个 Context。

```python
import vaststream

# 创建 Context
context = vaststream.Context(device_id=0)
```

### Graph

计算图，用于组织算子和模型的执行流程。

### Stream

Stream 是 Graph 的实例化，用于执行推理任务。

### VDSP

Vastai Deep Learning Streaming Processor，瀚博深度学习流处理器。

### 数据容器

#### DataHandle

数据句柄，管理 Device 侧的数据缓冲区。

```python
# 创建 DataHandle
handle = vaststream.DataHandle(context, size, memory_type)

# 获取数据指针
data_ptr = handle.get_ptr()

# 释放
handle.destroy()
```

#### DataBuffer

数据缓冲区，用于 Host 和 Device 之间的数据传输。

```python
# 创建 DataBuffer
buffer = vaststream.DataBuffer(context, size)

# 拷贝数据到 Device
buffer.copy_from_host(host_data)

# 从 Device 拷贝数据
host_data = buffer.copy_to_host()

# 释放
buffer.destroy()
```

#### Tensor

张量，用于存储多维数组数据。

```python
import numpy as np

# 从 numpy 数组创建 Tensor
np_array = np.random.randn(1, 3, 224, 224).astype(np.float32)
tensor = vaststream.Tensor.from_numpy(context, np_array)

# 转换为 numpy 数组
result_array = tensor.to_numpy()

# 获取 Tensor 信息
shape = tensor.shape
dtype = tensor.dtype
ndim = tensor.ndim
```

#### Dataset

数据集，用于批量数据处理。

```python
# 创建 Dataset
dataset = vaststream.Dataset(context)

# 添加 Tensor
dataset.add_tensor("input", input_tensor)
dataset.add_tensor("output", output_tensor)

# 获取 Tensor
input_tensor = dataset.get_tensor("input")
```

#### ModelDataset

模型数据集，专门用于模型推理的输入输出管理。

```python
# 创建 ModelDataset
model_dataset = vaststream.ModelDataset(context)

# 设置输入
model_dataset.set_input("input_name", input_tensor)

# 获取输出
output_tensor = model_dataset.get_output("output_name")
```

## 运行资源管理 (VACM)

### 日志配置

```python
import vaststream

# 设置日志级别
vaststream.set_log_level(vaststream.LOG_LEVEL_INFO)

# 设置日志输出路径
vaststream.set_log_path("/var/log/vaststream.log")
```

### 接口调用流程

#### 单线程单 Context

```python
import vaststream

# 初始化
vaststream.initialize()

# 创建 Context
context = vaststream.Context(device_id=0)

# 使用 Context 进行后续操作
# ...

# 销毁 Context
del context

# 反初始化
vaststream.uninitialize()
```

#### 单线程多 Context

```python
import vaststream

vaststream.initialize()

# 创建多个 Context
context1 = vaststream.Context(device_id=0)
context2 = vaststream.Context(device_id=0)  # 同一设备的多个 Context

# 在不同 Context 上执行操作
# ...

# 清理
del context1
del context2
vaststream.uninitialize()
```

#### 多线程单 Context

```python
import vaststream
import threading

vaststream.initialize()
context = vaststream.Context(device_id=0)

def worker(thread_id):
    # 增加 Context 引用
    context.retain()
    
    try:
        # 使用 Context
        # ...
        pass
    finally:
        # 减少 Context 引用
        context.release()

# 创建多个线程
threads = []
for i in range(4):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

# 等待线程完成
for t in threads:
    t.join()

# 清理
del context
vaststream.uninitialize()
```

#### 单线程多 Device

```python
import vaststream

vaststream.initialize()

# 获取设备数量
device_count = vaststream.get_device_count()

# 为每个设备创建 Context
contexts = []
for i in range(device_count):
    ctx = vaststream.Context(device_id=i)
    contexts.append(ctx)

# 在不同设备上执行操作
# ...

# 清理
for ctx in contexts:
    del ctx
vaststream.uninitialize()
```

## 图像运算操作 (VACE)

### 算子支持

#### 图像变换算子

- Resize: 图像缩放
- Crop: 图像裁剪
- Flip: 图像翻转
- CvtColor: 颜色空间转换
- Normalize: 归一化

#### 融合算子

- 组合预处理算子（Resize + CvtColor + Normalize）
- LetterBox: 保持比例的缩放填充

#### 自定义算子

支持加载用户自定义算子。

### 接口调用

```python
import vaststream

# 初始化
vaststream.vace_initialize()

# 创建 Resize 算子
resize_op = vaststream.ResizeOperator(
    width=224,
    height=224,
    interpolation=vaststream.INTERP_LINEAR
)

# 创建 CvtColor 算子
cvt_op = vaststream.CvtColorOperator(
    code=vaststream.COLOR_BGR2RGB
)

# 创建 Normalize 算子
norm_op = vaststream.NormalizeOperator(
    mean=[0.485, 0.456, 0.406],
    std=[0.229, 0.224, 0.225]
)

# 执行算子
output = resize_op.execute(input_tensor)
output = cvt_op.execute(output)
output = norm_op.execute(output)

# 释放
vaststream.vace_uninitialize()
```

## 推理应用开发 (VACL)

### 支持 Stream 类型

- 同步 Stream
- 异步 Stream
- 批量 Stream

### 接口调用流程

#### 1. 系统资源初始化

```python
import vaststream

# 初始化
vaststream.initialize()

# 创建 Context
context = vaststream.Context(device_id=0)
```

#### 2. 构建模型

```python
# 加载模型
model = vaststream.Model(
    context=context,
    model_path="/path/to/model.vam"
)

# 获取模型信息
input_shapes = model.get_input_shapes()
output_shapes = model.get_output_shapes()
input_dtypes = model.get_input_dtypes()
output_dtypes = model.get_output_dtypes()
```

#### 3. 构建 Graph

```python
# 创建 Graph
graph = vaststream.Graph(context)

# 创建根节点（输入节点）
input_node = graph.add_input(
    name="input",
    shape=(1, 3, 224, 224),
    dtype=vaststream.FLOAT32
)

# 添加预处理算子
resize_node = graph.add_operator(
    op=vaststream.ResizeOperator(224, 224),
    inputs=[input_node]
)

cvt_node = graph.add_operator(
    op=vaststream.CvtColorOperator(vaststream.COLOR_BGR2RGB),
    inputs=[resize_node]
)

norm_node = graph.add_operator(
    op=vaststream.NormalizeOperator(
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225]
    ),
    inputs=[cvt_node]
)

# 添加模型节点
model_node = graph.add_model(
    model=model,
    inputs=[norm_node]
)

# 添加输出节点
output_node = graph.add_output(
    name="output",
    shape=(1, 1000),
    dtype=vaststream.FLOAT32
)
```

#### 4. 构建 Stream

```python
# 创建 Stream
stream = vaststream.Stream(graph)

# 注册算子（获取中间结果）
stream.register_output(model_node, "model_output")

# 注册回调函数（异步模式）
def on_complete(stream, user_data):
    print("Stream execution completed!")
    # 获取输出
    output = stream.get_output("model_output")
    # 处理结果
    
stream.register_callback(on_complete, user_data=None)

# 构建 Stream
stream.build()
```

#### 5. 获取最少输入数据个数

```python
# 获取 Stream 所需的最少输入数量
min_inputs = stream.get_min_input_count()
print(f"Minimum inputs required: {min_inputs}")
```

#### 6. 构建模型的输入和输出容器

```python
import numpy as np

# 创建输入 Tensor
input_np = np.random.randn(1, 3, 224, 224).astype(np.float32)
input_tensor = vaststream.Tensor.from_numpy(context, input_np)

# 创建输出 Tensor
output_tensor = vaststream.Tensor(context, shape=(1, 1000), dtype=vaststream.FLOAT32)

# 或使用 ModelDataset
model_dataset = vaststream.ModelDataset(context)
model_dataset.set_input("input", input_tensor)
```

#### 7. 执行 Stream

##### 异步执行

```python
# 异步执行
stream.run_async(inputs=[input_tensor])

# 执行其他操作...

# 等待完成
stream.wait_until_done(timeout_ms=10000)

# 获取输出
output = stream.get_output("model_output")
result = output.to_numpy()
```

##### 同步执行

```python
# 同步执行
outputs = stream.run_sync(inputs=[input_tensor])

# 直接获取结果
result = outputs[0].to_numpy()
```

#### 8. 模型推理完整示例

```python
import vaststream
import numpy as np

# 初始化
vaststream.initialize()
context = vaststream.Context(device_id=0)

# 加载模型
model = vaststream.Model(context, "/path/to/resnet50.vam")

# 创建 Graph
graph = vaststream.Graph(context)
input_node = graph.add_input("input", (1, 3, 224, 224), vaststream.FLOAT32)

# 添加预处理
resize = graph.add_operator(
    vaststream.ResizeOperator(224, 224),
    [input_node]
)
cvt = graph.add_operator(
    vaststream.CvtColorOperator(vaststream.COLOR_BGR2RGB),
    [resize]
)
norm = graph.add_operator(
    vaststream.NormalizeOperator(
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225]
    ),
    [cvt]
)

# 添加模型
model_node = graph.add_model(model, [norm])
output_node = graph.add_output("output", (1, 1000), vaststream.FLOAT32)

# 创建 Stream
stream = vaststream.Stream(graph)
stream.register_output(model_node, "output")
stream.build()

# 准备输入
image = np.random.randn(224, 224, 3).astype(np.float32)
input_tensor = vaststream.Tensor.from_numpy(context, image)

# 执行推理
outputs = stream.run_sync([input_tensor])
result = outputs[0].to_numpy()

# 后处理（如 softmax）
probabilities = np.exp(result) / np.sum(np.exp(result))
predicted_class = np.argmax(probabilities)

print(f"Predicted class: {predicted_class}")

# 清理
del stream
del graph
del model
del context
vaststream.uninitialize()
```

## 媒体数据处理 (VAME)

### 文件解析

#### H.264/HEVC 文件解析

```python
import vaststream

# 创建视频文件解析器
parser = vaststream.VideoParser("/path/to/video.mp4")

# 获取视频信息
video_info = parser.get_info()
print(f"Width: {video_info.width}")
print(f"Height: {video_info.height}")
print(f"FPS: {video_info.fps}")
print(f"Codec: {video_info.codec}")

# 读取视频帧
while True:
    packet = parser.read_packet()
    if packet is None:
        break
    # 处理视频包
    
parser.close()
```

#### YUV 文件解析

```python
# 创建 YUV 文件解析器
parser = vaststream.YUVParser(
    "/path/to/video.yuv",
    width=1920,
    height=1080,
    format=vaststream.YUV420P
)

# 读取帧
while True:
    frame = parser.read_frame()
    if frame is None:
        break
    # 处理帧
    
parser.close()
```

### JPEG 解码 (JPEGD)

#### 同步解码

```python
import vaststream

# 创建 JPEG 解码器
decoder = vaststream.JpegDecoder(context)

# 读取 JPEG 文件
with open("/path/to/image.jpg", "rb") as f:
    jpeg_data = f.read()

# 同步解码
image = decoder.decode_sync(jpeg_data)

# 转换为 numpy
np_image = image.to_numpy()

# 获取图像信息
print(f"Width: {image.width}")
print(f"Height: {image.height}")
print(f"Format: {image.format}")
```

#### 异步解码

```python
# 创建 JPEG 解码器
decoder = vaststream.JpegDecoder(context)

# 启动解码器
decoder.start()

# 发送数据
decoder.send_data(jpeg_data)

# 接收解码后的图像
image = decoder.receive_image(timeout_ms=1000)
if image is not None:
    np_image = image.to_numpy()
    
# 停止解码器
decoder.stop()
```

### JPEG 编码 (JPEGE)

#### 同步编码

```python
# 创建 JPEG 编码器
encoder = vaststream.JpegEncoder(
    context,
    quality=85
)

# 准备图像数据
np_image = np.random.randint(0, 256, (1080, 1920, 3), dtype=np.uint8)
image = vaststream.Image.from_numpy(context, np_image)

# 同步编码
jpeg_data = encoder.encode_sync(image)

# 保存
with open("output.jpg", "wb") as f:
    f.write(jpeg_data)
```

#### 异步编码

```python
# 创建 JPEG 编码器
encoder = vaststream.JpegEncoder(context, quality=85)
encoder.start()

# 发送图像
encoder.send_image(image)

# 接收编码数据
jpeg_data = encoder.receive_data(timeout_ms=1000)

encoder.stop()
```

### 视频解码 (VDEC)

```python
# 创建视频解码器
decoder = vaststream.VideoDecoder(
    context,
    codec=vaststream.CODEC_H264,
    width=1920,
    height=1080
)

# 启动解码器
decoder.start()

# 读取视频文件并解码
parser = vaststream.VideoParser("/path/to/video.mp4")

while True:
    packet = parser.read_packet()
    if packet is None:
        break
        
    # 发送数据到解码器
    decoder.send_data(packet.data, packet.pts)
    
    # 尝试接收解码后的帧
    while True:
        frame = decoder.receive_frame(timeout_ms=0)
        if frame is None:
            break
        # 处理解码后的帧
        np_frame = frame.to_numpy()
        
# 刷新解码器
decoder.send_data(None)  # 发送 None 表示流结束

# 接收剩余帧
while True:
    frame = decoder.receive_frame(timeout_ms=1000)
    if frame is None:
        break
    # 处理帧

decoder.stop()
parser.close()
```

### 视频编码 (VENC)

```python
# 创建视频编码器
encoder = vaststream.VideoEncoder(
    context,
    codec=vaststream.CODEC_H264,
    width=1920,
    height=1080,
    fps=30,
    bitrate=4000000,
    rc_mode=vaststream.RC_VBR
)

# 启动编码器
encoder.start()

# 编码视频帧
frame_count = 0
while has_more_frames:
    # 获取帧数据
    np_frame = get_next_frame()
    frame = vaststream.Frame.from_numpy(context, np_frame)
    frame.pts = frame_count
    
    # 发送帧
    encoder.send_frame(frame)
    
    # 接收编码后的数据
    while True:
        packet = encoder.receive_packet(timeout_ms=0)
        if packet is None:
            break
        # 处理编码后的数据
        write_to_file(packet.data)
        
    frame_count += 1

# 刷新编码器
encoder.send_frame(None)

# 接收剩余数据
while True:
    packet = encoder.receive_packet(timeout_ms=1000)
    if packet is None:
        break
    write_to_file(packet.data)

encoder.stop()
```

## 设备资源管理 (VAML)

### 基础概念

- **Card**: 板卡，指瀚博 AI 加速器硬件
- **Die**: 芯片核心，一个 Card 可能包含多个 Die

### 核心功能

- 获取设备数量和基本信息
- 查询设备利用率、温度、功耗等
- 注册回调函数监控设备状态

### 接口调用

```python
import vaststream

# 初始化
vaststream.vaml_initialize()

# 获取板卡个数
card_count = vaststream.get_card_count()
print(f"Number of cards: {card_count}")

# 获取板卡句柄
card = vaststream.Card(card_index=0)

# 获取设备信息
card_info = card.get_info()
print(f"UUID: {card_info.uuid}")
print(f"Type: {card_info.card_type}")
print(f"PCIe Info: {card_info.pci_info}")

# 获取 Die 信息
die_count = card.get_die_count()
for i in range(die_count):
    die = card.get_die(i)
    die_info = die.get_info()
    
    print(f"Die {i}:")
    print(f"  Temperature: {die_info.temperature}°C")
    print(f"  Power: {die_info.power} mW")
    print(f"  Utilization: {die_info.utilization}%")
    print(f"  Memory Used: {die_info.memory_used} MB")
    print(f"  Memory Total: {die_info.memory_total} MB")

# 注册回调函数（监控设备状态）
def on_card_event(event_type, event_data, user_data):
    print(f"Card event: {event_type}")
    if event_type == vaststream.EVENT_TEMPERATURE_HIGH:
        print("Warning: Temperature is high!")

card.register_callback(on_card_event, user_data=None)

# 反注册
card.unregister_callback()

# 清理
del card
vaststream.vaml_uninitialize()
```

### 完整示例

```python
import vaststream
import time

# 初始化
vaststream.initialize()
vaststream.vaml_initialize()

# 获取所有设备信息
card_count = vaststream.get_card_count()
print(f"Found {card_count} card(s)")

for i in range(card_count):
    card = vaststream.Card(i)
    info = card.get_info()
    
    print(f"\nCard {i}:")
    print(f"  UUID: {info.uuid}")
    print(f"  Type: {info.card_type}")
    
    die_count = card.get_die_count()
    for j in range(die_count):
        die = card.get_die(j)
        die_info = die.get_info()
        
        print(f"  Die {j}:")
        print(f"    Temperature: {die_info.temperature}°C")
        print(f"    Power: {die_info.power / 1000:.2f} W")
        print(f"    Utilization: {die_info.utilization}%")
        print(f"    Memory: {die_info.memory_used}/{die_info.memory_total} MB")

# 持续监控
print("\nMonitoring for 10 seconds...")
def on_event(event_type, event_data, user_data):
    print(f"Event: {event_type}")

card = vaststream.Card(0)
card.register_callback(on_event)

time.sleep(10)

card.unregister_callback()
del card

# 清理
vaststream.vaml_uninitialize()
vaststream.uninitialize()
```

## 查看 SDK 版本

```python
import vaststream

# 查看 SDK 版本
print(vaststream.__version__)
```

## 注意事项

1. **资源管理**: Python SDK 会自动管理大部分资源，但仍建议显式调用 `del` 释放重要资源
2. **GIL 问题**: 长时间运行的操作会释放 GIL，支持真正的多线程并行
3. **内存管理**: 大内存操作注意及时释放，避免内存泄漏
4. **异常处理**: 使用 try-except 捕获 SDK 抛出的异常
5. **性能优化**:
   - 复用 Stream 和 Graph
   - 使用异步 API 提高吞吐量
   - 批量处理数据
   - 避免频繁的 Host-Device 数据传输

## 相关文档

- VastStream SDK 2.0 C++ 用户手册
- VastStreamX 用户手册
- VAMC 模型转换工具使用指南
