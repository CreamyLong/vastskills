---
name: "vastai-vaststreamx"
description: "瀚博半导体 VastStreamX C++ API 用户手册。Invoke when user needs to develop deep learning inference applications, video processing, or model deployment using VastStreamX SDK on Vastai hardware."
---

# VastStreamX 2.8.16 用户手册（C++）

VastStreamX 是 VastStream SDK 的更高层次封装，提供了 Stream 管理、内存管理、模型加载与执行、算子加载与执行、多媒体数据处理等 C++ API，用于实现目标识别、图像分类、视频增强等功能。

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

用于存储和管理 Device 侧的图像帧数据，一般作为 VastStreamX 内置算子的输入。

```cpp
// 创建 Image 对象
vsx::Image image;

// 从数据创建
image.AdoptData(data, width, height, format, colorspace);

// 从其他 Image 复制
image.CopyFrom(src_image);

// 克隆
vsx::Image cloned = image.Clone();
```

**主要方法**:
- `Format()`: 获取图像格式
- `ColorSpace()`: 获取颜色空间
- `Height()`: 获取图像高度
- `Width()`: 获取图像宽度
- `IsContiguous()`: 检查数据是否连续

### Tensor 类

用于存储和管理 Device 侧的张量数据，如存储 Stream 推理流程的结果。

```cpp
// 创建 Tensor
vsx::Tensor tensor(context, shape, dtype);

// 获取数据指针
void* data = tensor.GetDataAddress();

// 获取形状
std::vector<int64_t> shape = tensor.Shape();

// 获取数据类型
vsx::DataType dtype = tensor.GetDType();
```

**主要方法**:
- `ndim()`: 获取维度数
- `Shape()`: 获取形状
- `GetDataAddress()`: 获取数据地址
- `GetSize()`: 获取数据大小
- `CopyFrom()`: 从其他 Tensor 复制
- `Reshape()`: 重塑形状

### Model 类

用于加载和管理推理模型，支持静态和动态模型。

```cpp
// 创建静态模型
vsx::Model model(context, model_path);

// 创建动态模型
vsx::Model model(context, model_path, max_batch_size);

// 设置 batch size（动态模型）
model.SetBatchSize(batch_size);

// 获取输入/输出信息
int input_count = model.GetInputCount();
int output_count = model.GetOutputCount();
auto shape = model.GetInputShapeByIndex(0);
```

**主要方法**:
- `SetBatchSize()`: 设置 batch size
- `GetBatchSize()`: 获取当前 batch size
- `GetMaxBatchSize()`: 获取最大 batch size
- `GetInputCount()`: 获取输入数量
- `GetOutputCount()`: 获取输出数量
- `GetInputShapeByIndex()`: 获取输入形状
- `GetOutputShapeByIndex()`: 获取输出形状
- `IsDynamic()`: 是否为动态模型

### Graph 类

用于构建推理流程的计算图，将 Operator 和 Model 进行组合。

```cpp
// 创建 Graph
vsx::Graph graph;

// 添加输入算子
graph.AddInputOperator(input_op);

// 添加算子
graph.AddOperator(op1);
graph.AddOperator(op2);

// 批量添加算子
graph.AddOperators({op1, op2, op3});
```

**主要方法**:
- `AddInputOperator()`: 添加输入算子
- `AddOperator()`: 添加算子
- `AddOperators()`: 批量添加算子

### Stream 类

用于管理推理流程的执行，支持同步和异步执行。

```cpp
// 创建 Stream
vsx::Stream stream(context);

// 注册非模型算子输出
stream.RegisterNoneModelOperatorOutput(op, output_tensor);

// 注册模型算子输出
stream.RegisterModelOperatorOutput(model_op, output_tensors);

// 构建推理流程
stream.Build(graph);

// 同步执行
stream.RunSync(input_tensors, output_tensors);

// 异步执行
stream.RunAsync(input_tensors);
stream.WaitUntilDone();
```

**主要方法**:
- `RegisterNoneModelOperatorOutput()`: 注册非模型算子输出
- `RegisterModelOperatorOutput()`: 注册模型算子输出
- `Build()`: 构建推理流程
- `RunSync()`: 同步执行
- `RunAsync()`: 异步执行
- `GetOperatorOutput()`: 获取算子输出
- `WaitUntilDone()`: 等待异步执行完成
- `Cancel()`: 取消执行

### Operator 类

算子是深度学习算法中的基本计算单元。

#### BuildInOperator（内置算子）

```cpp
// 创建内置算子
vsx::BuildInOperator op(vsx::kSINGLE_OP_RESIZE);

// 设置属性
op.SetAttribute(vsx::kResizeHeight, 224);
op.SetAttribute(vsx::kResizeWidth, 224);
```

**常用内置算子类型**:
- `kSINGLE_OP_RESIZE`: 图像缩放
- `kSINGLE_OP_CROP`: 图像裁剪
- `kSINGLE_OP_FLIP`: 图像翻转
- `kSINGLE_OP_CVT_COLOR`: 颜色空间转换
- `kSINGLE_OP_NORM_TENSOR`: 张量归一化
- `kSINGLE_OP_SCALE`: 缩放
- `kSINGLE_OP_COPY_MAKE_BORDER`: 边界填充
- `kSINGLE_OP_BATCH_CROP_RESIZE`: 批量裁剪缩放

**融合算子**:
- `kFUSION_OP_YUV_NV12_RESIZE_CVTCOLOR_CROP_NORM_TENSOR`
- `kFUSION_OP_YUV_NV12_RESIZE_2RGB_NORM_TENSOR`
- `kFUSION_OP_YUV_NV12_LETTERBOX_2RGB_NORM_TENSOR`
- `kFUSION_OP_RGB_RESIZE_CVTCOLOR_NORM_TENSOR`
- `kFUSION_OP_RGB_LETTERBOX_CVTCOLOR_NORM_TENSOR`

#### ModelOperator（模型算子）

```cpp
// 从 Model 创建模型算子
vsx::ModelOperator model_op(model);
```

#### CustomOperator（自定义算子）

```cpp
// 创建自定义算子
vsx::CustomOperator custom_op("MyCustomOp");

// 设置回调函数
custom_op.SetCallback(callback);

// 设置配置
custom_op.SetConfig(config);
```

## 环境安装

### 硬件与系统要求

- CPU: Intel i5-4570 或更高配置，或性能相当的 AMD/ARM 架构 CPU
- Memory: 8GB 或以上
- 支持的卡类型：SV100 系列、SGPU100 系列

### Linux 环境安装

#### 1. 安装 PCIe 驱动

**Ubuntu 环境**:
```bash
# 安装驱动包
sudo dpkg -i vastai-pcie-driver_xxx.deb

# 加载驱动
sudo modprobe vastai_pcie

# 查看驱动版本
cat /sys/module/vastai_pcie/version
```

**CentOS 环境**:
```bash
# 安装驱动包
sudo rpm -ivh vastai-pcie-driver-xxx.rpm

# 加载驱动
sudo modprobe vastai_pcie
```

**卸载 PCIe 驱动**:
```bash
sudo modprobe -r vastai_pcie
sudo dpkg -r vastai-pcie-driver  # Ubuntu
sudo rpm -e vastai-pcie-driver   # CentOS
```

#### 2. 安装 VastStream SDK

```bash
# 解压 SDK
tar -xzvf VastStream-SDK-xxx.tar.gz

# 安装
sudo ./install.sh

# 设置环境变量
source /opt/vastai/vaststream/setup.sh
```

**卸载 VastStream SDK**:
```bash
sudo /opt/vastai/vaststream/uninstall.sh
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

**卸载 VastStreamX**:
```bash
sudo /opt/vastai/vaststreamx/uninstall.sh
```

### Windows 环境安装

1. 安装 PCIe 驱动
   - 运行驱动安装程序
   - 重启系统

2. 安装 AI Compiler
   - 运行安装程序
   - 设置环境变量

3. 安装 VastStream SDK 和 VastStreamX
   - 运行安装程序
   - 配置环境变量

## 快速入门：图像分类示例

### 开发流程

```cpp
#include <vaststreamx/vaststreamx.h>

int main() {
    // 1. 初始化系统
    vsx::InitSystem();
    
    // 2. 创建设备上下文
    vsx::Context context = vsx::Context::Create(0);  // 使用设备 0
    
    // 3. 加载模型
    vsx::Model model(context, "/path/to/resnet50.model");
    
    // 4. 创建模型算子
    vsx::ModelOperator model_op(model);
    
    // 5. 创建预处理算子
    vsx::BuildInOperator resize_op(vsx::kSINGLE_OP_RESIZE);
    resize_op.SetAttribute(vsx::kResizeHeight, 224);
    resize_op.SetAttribute(vsx::kResizeWidth, 224);
    
    vsx::BuildInOperator cvt_op(vsx::kSINGLE_OP_CVT_COLOR);
    cvt_op.SetAttribute(vsx::kCvtColorCode, vsx::kCOLOR_BGR2RGB);
    
    vsx::BuildInOperator norm_op(vsx::kSINGLE_OP_NORM_TENSOR);
    norm_op.SetAttribute(vsx::kNormType, vsx::kNORM_MEAN_STD);
    norm_op.SetAttribute(vsx::kNormMean, {0.485, 0.456, 0.406});
    norm_op.SetAttribute(vsx::kNormStd, {0.229, 0.224, 0.225});
    
    // 6. 构建计算图
    vsx::Graph graph;
    graph.AddInputOperator(resize_op);
    graph.AddOperator(cvt_op);
    graph.AddOperator(norm_op);
    graph.AddOperator(model_op);
    
    // 7. 创建 Stream 并构建
    vsx::Stream stream(context);
    stream.Build(graph);
    
    // 8. 准备输入数据
    vsx::Image input_image;
    input_image.AdoptData(image_data, width, height, 
                          vsx::kBGR, vsx::kCOLOR_SPACE_BT_601);
    
    std::vector<vsx::Tensor> inputs = {input_image};
    std::vector<vsx::Tensor> outputs;
    
    // 9. 执行推理
    stream.RunSync(inputs, outputs);
    
    // 10. 处理输出结果
    float* result = static_cast<float*>(outputs[0].GetDataAddress());
    
    // 11. 释放资源（自动）
    return 0;
}
```

### 编译应用

```bash
# 使用 CMake
cmake_minimum_required(VERSION 3.10)
project(my_app)

set(CMAKE_CXX_STANDARD 11)

# 查找 VastStreamX
find_package(VastStreamX REQUIRED)

add_executable(my_app main.cpp)
target_link_libraries(my_app VastStreamX::vaststreamx)
```

### 运行应用

```bash
# 设置环境变量
source /opt/vastai/vaststreamx/setup.sh

# 运行
./my_app
```

## 同步推理 vs 异步推理

### 同步推理

```cpp
vsx::Stream stream(context);
stream.Build(graph);

// 同步执行，阻塞直到完成
stream.RunSync(inputs, outputs);

// 直接使用输出结果
ProcessOutput(outputs);
```

### 异步推理

```cpp
vsx::Stream stream(context);
stream.Build(graph);

// 异步执行，立即返回
stream.RunAsync(inputs);

// 执行其他任务...

// 等待完成
stream.WaitUntilDone();

// 获取输出
stream.GetOperatorOutput(model_op, outputs);

ProcessOutput(outputs);
```

## 视频处理

### VideoCapture（视频捕获）

```cpp
vsx::VideoCapture capture;

// 打开视频文件
capture.open("/path/to/video.mp4");

// 检查是否成功打开
if (!capture.isOpened()) {
    std::cerr << "Failed to open video" << std::endl;
    return -1;
}

// 读取帧
vsx::Image frame;
while (capture.read(frame)) {
    // 处理帧
    ProcessFrame(frame);
}

// 释放
capture.release();
```

### VideoDecoder（视频解码）

```cpp
vsx::VideoDecoder decoder;

// 创建解码器
vsx::VideoDecoder decoder(codec_type);

// 发送编码数据
decoder.SendData(packet_data, packet_size);

// 接收解码后的图像
vsx::Image image;
while (decoder.RecvImage(image)) {
    // 处理图像
    ProcessImage(image);
}

// 停止发送
decoder.StopSendData();
```

### VideoEncoder（视频编码）

```cpp
vsx::VideoEncoder encoder;

// 配置编码参数
vsx::VideoEncoderAdvancedConfig config;
config.codec_type = vsx::kH264;
config.width = 1920;
config.height = 1080;
config.fps = 30;
config.bitrate = 4000000;

// 创建编码器
vsx::VideoEncoder encoder(config);

// 发送图像
encoder.SendImage(image);

// 接收编码数据
std::vector<uint8_t> data;
while (encoder.RecvData(data)) {
    // 处理编码数据
    ProcessEncodedData(data);
}

// 停止发送
encoder.StopSendImage();
```

## 硬件信息查询

### Card 类

```cpp
// 获取所有卡
std::vector<vsx::Card> cards = vsx::GetAllCards();

for (auto& card : cards) {
    // 获取 UUID
    std::string uuid = card.GetUUID();
    
    // 获取卡类型
    vsx::CardType type = card.GetCardType();
    
    // 获取 PCIe 信息
    vsx::CardPciInfo pci_info = card.GetPciInfo();
    
    // 获取功耗
    vsx::CardPower power = card.GetCardPower();
    
    // 获取风扇速度
    int fan_speed = card.GetFanSpeedLevel();
    
    // 获取 Die 数量
    int die_count = card.GetDieCount();
}
```

### Die 类

```cpp
// 获取 Die
vsx::Die die = card.GetDieByIndex(0);

// 获取时钟频率
vsx::DieClockFrequency freq = die.GetClockFrequency();

// 获取温度
vsx::DieTemperature temp = die.GetTemperature();

// 获取功耗
vsx::DevicePower power = die.GetPower();

// 获取内存使用
vsx::DieMemory memory = die.GetMemory();

// 获取利用率
vsx::DieUtilization util = die.GetUtilization();

// 获取运行中的进程
std::vector<vsx::DieProcessInfo> procs = die.GetRunningProcesses();
```

## 内存管理

### Device 内存分配

```cpp
// 分配 Device 内存
void* dev_ptr = vsx::MallocDevice(context, size);

// 分配扩展 Device 内存
void* dev_ptr_ext = vsx::MallocDeviceExt(context, size, flags);

// 释放 Device 内存
vsx::FreeDevice(context, dev_ptr);
vsx::FreeDeviceExt(context, dev_ptr_ext);
```

### Host 内存分配

```cpp
// 分配 Host 内存
void* host_ptr = vsx::MallocHost(context, size);

// 释放 Host 内存
vsx::FreeHost(context, host_ptr);
```

### 内存拷贝

```cpp
// Host to Device
vsx::Memcpy(context, dev_ptr, host_ptr, size, vsx::kHOST_TO_DEVICE);

// Device to Host
vsx::Memcpy(context, host_ptr, dev_ptr, size, vsx::kDEVICE_TO_HOST);

// Device to Device
vsx::MemcpyDevice(context, dst_ptr, src_ptr, size);

// 设置内存
vsx::MemsetDevice(context, dev_ptr, value, size);
```

## 常用数据结构

### StreamBalanceMode

```cpp
enum StreamBalanceMode {
    kBALANCE_MODE_NONE = 0,      // 不启用负载均衡
    kBALANCE_MODE_ROUND_ROBIN,   // 轮询模式
    kBALANCE_MODE_LEAST_LOAD     // 最小负载模式
};
```

### BuildInOperatorType

常用算子类型：
- 单算子：`kSINGLE_OP_*`
- 融合算子：`kFUSION_OP_*`

### ImageFormat

```cpp
enum ImageFormat {
    kBGR = 0,
    kRGB,
    kNV12,
    kNV21,
    kYUV420P,
    kGRAY,
    // ...
};
```

### DataType

```cpp
enum DataType {
    kINT8 = 0,
    kUINT8,
    kINT16,
    kUINT16,
    kINT32,
    kUINT32,
    kFLOAT16,
    kFLOAT32,
    // ...
};
```

## 注意事项

1. **资源管理**: VastStreamX 会自动管理资源，对象销毁时会自动释放资源
2. **线程安全**: Stream 对象不是线程安全的，每个线程应使用独立的 Stream
3. **内存对齐**: Device 内存分配会自动对齐，Host 内存建议使用页对齐
4. **错误处理**: 使用 try-catch 捕获异常进行错误处理
5. **性能优化**: 
   - 使用融合算子减少数据传输
   - 合理设置 batch size
   - 使用异步执行提高吞吐量

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 模型加载失败 | 检查模型路径和格式是否正确 |
| 内存分配失败 | 检查设备内存是否充足 |
| 推理结果异常 | 检查预处理参数是否正确 |
| 性能不达标 | 使用异步执行，调整 batch size |
| 设备无法识别 | 检查 PCIe 驱动是否正确安装 |

## 相关文档

- VAML API 参考手册
- VAMP 工具使用指南
- Vasmi 工具使用指南
- Vaprofiler 工具使用指南
