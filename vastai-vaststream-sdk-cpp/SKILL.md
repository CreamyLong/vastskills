---
name: "vastai-vaststream-sdk-cpp"
description: "瀚博半导体 VastStream SDK 2.0 C++ 用户手册。Invoke when user needs to develop deep learning inference and video codec applications using VastStream SDK C++ API on Vastai hardware."
---

# VastStream SDK 2.0 用户手册 (C++ 版)

VastStream SDK 是一套用于在瀚博设备平台上开发深度神经网络推理及视频编解码应用的 API 库，提供运行资源管理、内存管理、模型加载与执行、算子加载与执行、媒体数据处理等 API。

## 软件栈架构

VastStream 软件栈由以下五个核心功能模块组成：

- **VACM** (Vastai Common Library): 基础功能、日志、设备上下文、数据缓冲、数据集、张量、内存管理等跨平台封装
- **VACE** (Vastai Computing Engine): 算子库，支持图像变换、融合算子、自定义算子
- **VACL** (Vastai Accelerate Language): 加载 AI 模型、构建 VDSP Stream、运行 Stream
- **VAME** (Vastai Media Engine): JPEG 图片及视频编解码
- **VAML** (Vastai Management Library): 设备信息管理接口

## 基本概念

### 同步/异步 (Sync/Async)

- **同步**: 接口调用后立即执行，阻塞直到完成
- **异步**: 接口调用后立即返回，通过回调函数或事件通知完成

### 主机与设备

- **Host**: 主机端，指 CPU 和内存
- **Device**: 设备端，指瀚博 AI 加速器

### 设备上下文 (Context)

Context 是设备资源的抽象，用于管理设备内存、Stream 等资源。一个 Device 可以创建多个 Context，但一个 Context 只能属于一个 Device。

## VACM 模块 (通用库)

### 核心功能

- 日志管理 (vslog)
- 设备及其上下文管理 (Device & Context)
- 数据缓冲管理 (DataBuffer)
- 数据集管理 (Dataset)
- 张量 (Tensor)
- 内存管理 (Memory)
- 操作系统调用封装 (Event 和 Mutex)

### 接口调用流程

#### 单线程单 Context

```cpp
#include <vacm/vacm.h>

// 1. 初始化
vacmInitialize();

// 2. 创建设备上下文
vacmContext_t context;
vacmCreateContext(&context, device_id);

// 3. 使用 Context 进行后续操作
// ...

// 4. 销毁 Context
vacmDestroyContext(context);

// 5. 反初始化
vacmUninitialize();
```

#### 多线程单 Context

```cpp
// 主线程
vacmInitialize();
vacmCreateContext(&context, device_id);

// 子线程 1
vacmRetainContext(context);  // 增加引用计数
// 使用 context...
vacmReleaseContext(context); // 减少引用计数

// 子线程 2
vacmRetainContext(context);
// 使用 context...
vacmReleaseContext(context);

// 主线程清理
vacmDestroyContext(context);
vacmUninitialize();
```

### 数据容器

#### DataHandle

数据句柄，用于管理 Device 侧的数据缓冲区。

```cpp
vacmDataHandle_t handle;
vacmCreateDataHandle(&handle, context, size, memory_type);

// 获取数据指针
void* data = vacmGetDataHandlePtr(handle);

// 释放
vacmDestroyDataHandle(handle);
```

#### DataBuffer

数据缓冲区，用于 Host 和 Device 之间的数据传输。

```cpp
vacmDataBuffer_t buffer;
vacmCreateDataBuffer(&buffer, context, size);

// 拷贝数据到 Device
vacmCopyDataBufferToDevice(buffer, host_data, size);

// 从 Device 拷贝数据
vacmCopyDataBufferFromDevice(host_data, buffer, size);

// 释放
vacmDestroyDataBuffer(buffer);
```

#### Tensor

张量，用于存储多维数组数据。

```cpp
// 创建 Tensor
vacmTensor_t tensor;
int dims[] = {1, 3, 224, 224};
vacmCreateTensor(&tensor, context, dims, 4, VACM_DATA_TYPE_FLOAT32);

// 获取 Tensor 信息
void* data = vacmGetTensorData(tensor);
int* shape = vacmGetTensorShape(tensor);
int ndim = vacmGetTensorNDim(tensor);

// 释放
vacmDestroyTensor(tensor);
```

#### Dataset

数据集，用于批量数据处理。

```cpp
vacmDataset_t dataset;
vacmCreateDataset(&dataset, context);

// 添加数据
vacmDatasetAddTensor(dataset, "input", input_tensor);
vacmDatasetAddTensor(dataset, "output", output_tensor);

// 获取数据
vacmTensor_t tensor = vacmDatasetGetTensor(dataset, "input");

// 释放
vacmDestroyDataset(dataset);
```

## VACE 模块 (图像运算引擎)

### 算子支持

#### 图像变换算子

- Resize: 图像缩放
- Crop: 图像裁剪
- Flip: 图像翻转
- Rotate: 图像旋转
- CvtColor: 颜色空间转换
- Normalize: 归一化

#### 融合算子

- Resize + CvtColor + Normalize: 组合预处理
- LetterBox: 保持比例的缩放填充

#### 自定义算子

支持加载用户自定义的算子实现。

### 接口调用

```cpp
#include <vace/vace.h>

// 初始化
vaceInitialize();

// 创建算子
vaceOperator_t resize_op;
vaceCreateOperator(&resize_op, VACE_OP_RESIZE);

// 设置参数
vaceSetOperatorAttr(resize_op, "width", 224);
vaceSetOperatorAttr(resize_op, "height", 224);
vaceSetOperatorAttr(resize_op, "interpolation", VACE_INTERP_LINEAR);

// 执行算子
vaceExecuteOperator(resize_op, input_tensor, output_tensor);

// 释放
vaceDestroyOperator(resize_op);
vaceUninitialize();
```

## VACL 模块 (AI 推理运算库)

### 核心概念

#### Graph

计算图，用于组织算子和模型的执行流程。

```cpp
#include <vacl/vacl.h>

// 创建 Graph
vaclGraph_t graph;
vaclCreateGraph(&graph, context);

// 添加算子节点
vaclNodeId_t input_node = vaclGraphAddInput(graph, "input", input_desc);
vaclNodeId_t preprocess_node = vaclGraphAddOperator(graph, preprocess_op, &input_node, 1);
vaclNodeId_t model_node = vaclGraphAddModel(graph, model, &preprocess_node, 1);
vaclNodeId_t output_node = vaclGraphAddOutput(graph, "output", output_desc);
```

#### Stream

Stream 是 Graph 的实例化，用于执行推理任务。

```cpp
// 创建 Stream
vaclStream_t stream;
vaclCreateStream(&stream, graph);

// 注册输出
vaclStreamRegisterOutput(stream, output_node, output_tensor);

// 构建 Stream
vaclStreamBuild(stream);
```

#### Run Model 算子

用于在 Stream 中执行模型推理。

```cpp
// 加载模型
vaclModel_t model;
vaclLoadModel(&model, context, "/path/to/model.vam");

// 创建模型算子
vaclOperator_t model_op;
vaclCreateModelOperator(&model_op, model);
```

### 推理应用开发流程

#### 1. 系统资源初始化

```cpp
// 初始化 VACM
vacmInitialize();

// 创建设备上下文
vacmContext_t context;
vacmCreateContext(&context, 0);  // 使用设备 0
```

#### 2. 构建 Graph

```cpp
// 创建 Graph
vaclGraph_t graph;
vaclCreateGraph(&graph, context);

// 创建并添加算子
// 输入节点
vaclTensorDesc_t input_desc = {...};
vaclNodeId_t input_node = vaclGraphAddInput(graph, "input", input_desc);

// 预处理算子
vaceOperator_t resize_op;
vaceCreateOperator(&resize_op, VACE_OP_RESIZE);
vaceSetOperatorAttr(resize_op, "width", 224);
vaceSetOperatorAttr(resize_op, "height", 224);
vaclNodeId_t resize_node = vaclGraphAddOperator(graph, resize_op, &input_node, 1);

// 颜色转换算子
vaceOperator_t cvt_op;
vaceCreateOperator(&cvt_op, VACE_OP_CVT_COLOR);
vaceSetOperatorAttr(cvt_op, "code", VACE_COLOR_BGR2RGB);
vaclNodeId_t cvt_node = vaclGraphAddOperator(graph, cvt_op, &resize_node, 1);

// 归一化算子
vaceOperator_t norm_op;
vaceCreateOperator(&norm_op, VACE_OP_NORMALIZE);
float mean[] = {0.485, 0.456, 0.406};
float std[] = {0.229, 0.224, 0.225};
vaceSetOperatorAttr(norm_op, "mean", mean, 3);
vaceSetOperatorAttr(norm_op, "std", std, 3);
vaclNodeId_t norm_node = vaclGraphAddOperator(graph, norm_op, &cvt_node, 1);

// 模型节点
vaclModel_t model;
vaclLoadModel(&model, context, "/path/to/resnet50.vam");
vaclNodeId_t model_node = vaclGraphAddModel(graph, model, &norm_node, 1);

// 输出节点
vaclTensorDesc_t output_desc = {...};
vaclNodeId_t output_node = vaclGraphAddOutput(graph, "output", output_desc);
```

#### 3. 构建 Stream

```cpp
// 创建 Stream
vaclStream_t stream;
vaclCreateStream(&stream, graph);

// 注册需要获取输出结果的算子
vaclStreamRegisterOutput(stream, model_node, output_tensor);

// 构建 Stream
vaclStreamBuild(stream);
```

#### 4. 执行 Stream

##### 同步执行

```cpp
// 准备输入数据
vacmTensor_t input_tensor;
// ... 填充输入数据

// 执行 Stream
vaclStreamRunSync(stream, &input_tensor, 1, &output_tensor, 1);

// 处理输出结果
// ...
```

##### 异步执行

```cpp
// 定义回调函数
void OnStreamComplete(vaclStream_t stream, void* user_data) {
    // 获取输出结果
    vacmTensor_t output;
    vaclStreamGetOutput(stream, "output", &output);
    
    // 处理结果
    // ...
}

// 注册回调
vaclStreamRegisterCallback(stream, OnStreamComplete, user_data);

// 异步执行
vaclStreamRunAsync(stream, &input_tensor, 1);

// 等待完成（可选）
vaclStreamWait(stream, timeout);
```

## VAME 模块 (媒体处理引擎)

### JPEG 解码 (JPEGD)

#### 同步解码

```cpp
#include <vame/vame.h>

// 初始化
vameInitialize();

// 获取 JPEG 信息
vameJpegInfo_t jpeg_info;
vameGetJpegInfo(&jpeg_info, jpeg_data, jpeg_size);

// 创建解码通道
vameDecChannel_t channel;
vameDecChannelParam_t param = {
    .codec_type = VAME_CODEC_JPEG,
    .pixel_format = VAME_PIXEL_FORMAT_RGB,
    // ...
};
vameCreateDecoderChannel(&channel, context, &param);

// 同步解码
vameFrame_t output_frame;
vameJpegSyncDecode(channel, jpeg_data, jpeg_size, &output_frame);

// 使用解码后的图像
// ...

// 释放帧
vameDecReleaseFrame(channel, &output_frame);

// 销毁通道
vameDestroyDecoderChannel(channel);
```

#### 异步解码

```cpp
// 创建解码通道
vameCreateDecoderChannel(&channel, context, &param);

// 启动解码器
vameStartDecoder(channel);

// 发送数据
vameStream_t stream = {
    .data = jpeg_data,
    .size = jpeg_size,
    // ...
};
vameSendStreamToDecoder(channel, &stream);

// 接收解码后的帧
vameFrame_t frame;
while (vameReceiveFrameFromDecoder(channel, &frame, timeout) == VAME_SUCCESS) {
    // 处理帧
    // ...
    
    // 释放帧
    vameDecReleaseFrame(channel, &frame);
}

// 停止解码器
vameStopDecoder(channel);
vameDestroyDecoderChannel(channel);
```

### JPEG 编码 (JPEGE)

#### 同步编码

```cpp
// 创建编码通道
vameEncChannel_t channel;
vameEncChannelParam_t param = {
    .codec_type = VAME_CODEC_JPEG,
    .quality = 85,
    // ...
};
vameCreateEncoderChannel(&channel, context, &param);

// 同步编码
vameStream_t output_stream;
vameSendFrameToEncoderSync(channel, &input_frame, &output_stream);

// 使用编码后的数据
// ...

// 释放流
vameEncReleaseStream(channel, &output_stream);

// 销毁通道
vameDestroyEncoderChannel(channel);
```

### 视频解码 (VDEC)

```cpp
// 创建视频解码通道
vameDecChannelParam_t param = {
    .codec_type = VAME_CODEC_H264,  // 或 VAME_CODEC_HEVC
    // ...
};
vameCreateDecoderChannel(&channel, context, &param);

// 启动解码器
vameStartDecoder(channel);

// 循环发送视频流并接收解码后的帧
while (has_more_data) {
    // 发送编码数据
    vameStream_t stream = {
        .data = video_packet,
        .size = packet_size,
        .pts = pts,
        // ...
    };
    vameSendStreamToDecoder(channel, &stream);
    
    // 尝试接收解码后的帧
    vameFrame_t frame;
    while (vameReceiveFrameFromDecoder(channel, &frame, 0) == VAME_SUCCESS) {
        // 处理解码后的帧
        ProcessFrame(&frame);
        vameDecReleaseFrame(channel, &frame);
    }
}

// 刷新解码器
vameSendStreamToDecoder(channel, NULL);  // 发送 NULL 表示流结束

// 接收剩余的帧
vameFrame_t frame;
while (vameReceiveFrameFromDecoder(channel, &frame, timeout) == VAME_SUCCESS) {
    ProcessFrame(&frame);
    vameDecReleaseFrame(channel, &frame);
}

// 停止并销毁
vameStopDecoder(channel);
vameDestroyDecoderChannel(channel);
```

### 视频编码 (VENC)

```cpp
// 创建视频编码通道
vameEncVideoConfig_t config = {
    .codec_type = VAME_CODEC_H264,
    .width = 1920,
    .height = 1080,
    .fps = 30,
    .bitrate = 4000000,
    .rc_mode = VAME_RC_VBR,
    // ...
};

vameEncChannelParam_t param = {
    .codec_type = VAME_CODEC_H264,
    .video_config = &config,
    // ...
};
vameCreateEncoderChannel(&channel, context, &param);

// 启动编码器
vameStartEncoder(channel);

// 循环发送帧并接收编码后的数据
while (has_more_frames) {
    // 发送帧
    vameFrame_t frame = {
        .data = {y_data, u_data, v_data},
        .pts = pts,
        // ...
    };
    vameSendFrameToEncoder(channel, &frame);
    
    // 接收编码后的数据
    vameStream_t stream;
    while (vameReceiveStreamFromEncoder(channel, &stream, 0) == VAME_SUCCESS) {
        // 处理编码后的数据
        ProcessEncodedData(&stream);
        vameEncReleaseStream(channel, &stream);
    }
}

// 刷新编码器
vameSendFrameToEncoder(channel, NULL);

// 接收剩余的数据
vameStream_t stream;
while (vameReceiveStreamFromEncoder(channel, &stream, timeout) == VAME_SUCCESS) {
    ProcessEncodedData(&stream);
    vameEncReleaseStream(channel, &stream);
}

// 停止并销毁
vameStopEncoder(channel);
vameDestroyEncoderChannel(channel);
```

## VAML 模块 (设备管理库)

### 核心功能

- 获取设备列表
- 查询设备信息（温度、功耗、利用率等）
- 设备监控回调

### 接口调用

```cpp
#include <vaml/vaml.h>

// 初始化
vamlInitialize();

// 获取板卡个数
unsigned int card_count;
vamlGetCardCount(&card_count);

// 获取板卡句柄
vamlCard_t card;
vamlGetCardHandle(&card, card_index);

// 获取设备信息
vamlCardInfo_t card_info;
vamlGetCardInfo(card, &card_info);
printf("Card UUID: %s\n", card_info.uuid);
printf("Card Type: %d\n", card_info.card_type);

// 获取 Die 信息
unsigned int die_count;
vamlGetDieCount(card, &die_count);

for (unsigned int i = 0; i < die_count; i++) {
    vamlDie_t die;
    vamlGetDieHandle(card, i, &die);
    
    vamlDieInfo_t die_info;
    vamlGetDieInfo(die, &die_info);
    
    printf("Die %d Temperature: %d°C\n", i, die_info.temperature);
    printf("Die %d Power: %d mW\n", i, die_info.power);
    printf("Die %d Utilization: %d%%\n", i, die_info.utilization);
}

// 注册回调函数（用于监控）
vamlRegisterCallback(card, OnCardEvent, user_data);

// 反注册
vamlUnregisterCallback(card);

// 释放句柄
vamlReleaseCardHandle(card);

// 反初始化
vamlUninitialize();
```

## 注意事项

1. **资源管理**: 遵循"谁创建谁销毁"原则，确保所有创建的资源都被正确释放
2. **线程安全**: VACM Context 不是线程安全的，多线程访问需要加锁或使用独立的 Context
3. **内存对齐**: Device 内存分配有对齐要求，建议使用 SDK 提供的内存分配接口
4. **错误处理**: 所有 API 都返回状态码，应该检查返回值处理错误
5. **性能优化**:
   - 复用 Stream 和 Graph，避免重复创建
   - 使用异步执行提高吞吐量
   - 合理设置 batch size
   - 使用零拷贝技术减少数据传输

## 相关文档

- VastStreamX 用户手册（C++）
- VAML API 参考手册
- VAME API 参考手册
- VAMC 模型转换工具使用指南
