---
name: "vastai-vame-api"
description: "瀚博半导体 VAME API 2.8.3 参考手册。Invoke when user needs to develop video/image codec applications using VAME (Vastai Media Engine) C API on Vastai hardware."
---

# 媒体数据处理 API 2.8.3 参考手册

VAME (Vastai Media Engine，瀚博媒体处理引擎) 为第三方应用提供图像、视频处理的通用接口平台，支持媒体数据处理系统底层初始化、编解码操作等功能。

## 概述

VAME API 提供以下核心功能：

- **视频/图像解码**: JPEG、H.264、HEVC 等格式的硬件解码
- **视频/图像编码**: JPEG、H.264、HEVC 等格式的硬件编码
- **设备管理**: Device 设置、重置、获取等
- **系统配置**: 版本获取、错误描述等

### 函数指针调用方式

VAME API 中定义的所有 `FN_{function}` 类型函数指针均指向其对应的函数接口 `{function}`。这些函数接口只能通过函数指针调用，用户必须通过调用特定的实例创建接口以获取这些函数指针：

- **通用功能**: 通过 `vameCommonAPICreateInstance` 接口获取
- **视频/图像解码**: 通过 `vameDecodeAPICreateInstance` 接口获取
- **视频/图像编码**: 通过 `vameEncodeAPICreateInstance` 接口获取

## 通用接口

### 系统配置

#### FN_vameGetErrDesc

获取错误描述信息。

```c
const char* vameGetErrDesc(VAME_ERR err);
```

#### FN_vameGetVersion

获取 VAME 版本信息。

```c
VAME_ERR vameGetVersion(VAME_Version* version);
```

#### FN_vameCommonAPICreateInstance

创建通用 API 实例，获取通用功能函数指针。

```c
VAME_ERR vameCommonAPICreateInstance(
    VAME_CommonFunctionList* func_list
);
```

### 初始化与反初始化

#### FN_vameInitialize

初始化 VAME 系统。

```c
VAME_ERR vameInitialize(const VAME_InitParams* params);
```

#### FN_vameUninitialize

反初始化 VAME 系统。

```c
VAME_ERR vameUninitialize(void);
```

#### FN_vameSystemInitialize

系统级初始化。

```c
VAME_ERR vameSystemInitialize(void);
```

#### FN_vameSystemUninitialize

系统级反初始化。

```c
VAME_ERR vameSystemUninitialize(void);
```

### Device 管理

#### FN_vameSetDevice

设置当前设备。

```c
VAME_ERR vameSetDevice(VAME_Device device);
```

#### FN_vameResetDevice

重置设备。

```c
VAME_ERR vameResetDevice(VAME_Device device);
```

#### FN_vameGetDevice

获取当前设备。

```c
VAME_ERR vameGetDevice(VAME_Device* device);
```

## 视频/图像解码接口

### 实例创建

#### FN_vameDecodeAPICreateInstance

创建解码 API 实例，获取解码功能函数指针。

```c
VAME_ERR vameDecodeAPICreateInstance(
    VAME_DecodeApiFunctionList* func_list
);
```

### 通道管理

#### FN_vameCreateDecoderChannel

创建解码通道。

```c
VAME_ERR vameCreateDecoderChannel(
    VAME_ChId* channel_id,
    const VAME_DecChannelParameters* params
);
```

参数说明：
- `channel_id`: 输出参数，返回创建的通道 ID
- `params`: 解码通道参数配置

#### FN_vameDestroyDecoderChannel

销毁解码通道。

```c
VAME_ERR vameDestroyDecoderChannel(VAME_ChId channel_id);
```

### 信息获取

#### FN_vameGetVideoInfo

获取视频信息。

```c
VAME_ERR vameGetVideoInfo(
    const VAME_Stream* stream,
    VAME_VideoInfo* info
);
```

#### FN_vameGetJpegInfo

获取 JPEG 图像信息。

```c
VAME_ERR vameGetJpegInfo(
    const void* data,
    size_t size,
    VAME_DecJpegInfo* info
);
```

#### FN_vameJpegDecGetCaps

获取 JPEG 解码能力。

```c
VAME_ERR vameJpegDecGetCaps(VAME_JpegDecCapability* caps);
```

#### FN_vameVideoDecGetCaps

获取视频解码能力。

```c
VAME_ERR vameVideoDecGetCaps(VAME_VideoDecCapability* caps);
```

### 解码控制

#### FN_vameStartDecoder

启动解码器。

```c
VAME_ERR vameStartDecoder(VAME_ChId channel_id);
```

#### FN_vameResetDecoder

重置解码器。

```c
VAME_ERR vameResetDecoder(VAME_ChId channel_id);
```

#### FN_vameStopDecoder

停止解码器。

```c
VAME_ERR vameStopDecoder(VAME_ChId channel_id);
```

#### FN_vameSendStreamToDecoder

发送流数据到解码器。

```c
VAME_ERR vameSendStreamToDecoder(
    VAME_ChId channel_id,
    const VAME_Stream* stream
);
```

#### FN_vameJpegSyncDecoder

JPEG 同步解码。

```c
VAME_ERR vameJpegSyncDecode(
    VAME_ChId channel_id,
    const void* data,
    size_t size,
    VAME_Frame* frame
);
```

### 数据接收

#### FN_vameTransferFrameFromDecoder

从解码器传输帧。

```c
VAME_ERR vameTransferFrameFromDecoder(
    VAME_ChId channel_id,
    VAME_Frame* frame
);
```

#### FN_vameReceiveFrameFromDecoder

从解码器接收帧。

```c
VAME_ERR vameReceiveFrameFromDecoder(
    VAME_ChId channel_id,
    VAME_Frame* frame,
    int timeout_ms
);
```

#### FN_vameGetStreamInfoFromDecoder

从解码器获取流信息。

```c
VAME_ERR vameGetStreamInfoFromDecoder(
    VAME_ChId channel_id,
    VAME_StreamInfo* info
);
```

### 状态查询

#### FN_vameGetDecoderStatus

获取解码器状态。

```c
VAME_ERR vameGetDecoderStatus(
    VAME_ChId channel_id,
    VAME_Status* status
);
```

#### FN_vameGetDecoderAvailableChannels

获取可用解码通道数。

```c
VAME_ERR vameGetDecoderAvailableChannels(int* count);
```

#### FN_vameDecReleaseFrame

释放解码帧。

```c
VAME_ERR vameDecReleaseFrame(
    VAME_ChId channel_id,
    VAME_Frame* frame
);
```

#### FN_vameDecGetIdleDpbBufferCount

获取空闲 DPB 缓冲区数量。

```c
VAME_ERR vameDecGetIdleDpbBufferCount(
    VAME_ChId channel_id,
    int* count
);
```

## 视频/图像编码接口

### 实例创建

#### FN_vameEncodeAPICreateInstance

创建编码 API 实例，获取编码功能函数指针。

```c
VAME_ERR vameEncodeAPICreateInstance(
    VAME_EncodeApiFunctionList* func_list
);
```

### 通道管理

#### FN_vameCreateEncoderChannel

创建编码通道。

```c
VAME_ERR vameCreateEncoderChannel(
    VAME_ChId* channel_id,
    const VAME_EncChannelParameters* params
);
```

#### FN_vameDestroyEncoderChannel

销毁编码通道。

```c
VAME_ERR vameDestroyEncoderChannel(VAME_ChId channel_id);
```

### 编码控制

#### FN_vameSetEncoderVfMode

设置编码器视频场模式。

```c
VAME_ERR vameSetEncoderVfMode(
    VAME_ChId channel_id,
    VAME_EncVfMode mode
);
```

#### FN_vameStartEncoder

启动编码器。

```c
VAME_ERR vameStartEncoder(VAME_ChId channel_id);
```

#### FN_vameResetEncoder

重置编码器。

```c
VAME_ERR vameResetEncoder(VAME_ChId channel_id);
```

#### FN_vameStopEncoder

停止编码器。

```c
VAME_ERR vameStopEncoder(VAME_ChId channel_id);
```

#### FN_vameSendFrameToEncoder

发送帧到编码器。

```c
VAME_ERR vameSendFrameToEncoder(
    VAME_ChId channel_id,
    const VAME_Frame* frame
);
```

#### FN_vameSendFrameToEncoderSync

同步发送帧到编码器并获取编码数据。

```c
VAME_ERR vameSendFrameToEncoderSync(
    VAME_ChId channel_id,
    const VAME_Frame* frame,
    VAME_Stream* stream
);
```

### 数据接收

#### FN_vameReceiveStreamFromEncoder

从编码器接收流数据。

```c
VAME_ERR vameReceiveStreamFromEncoder(
    VAME_ChId channel_id,
    VAME_Stream* stream,
    int timeout_ms
);
```

#### FN_vameEncReleaseStream

释放编码流。

```c
VAME_ERR vameEncReleaseStream(
    VAME_ChId channel_id,
    VAME_Stream* stream
);
```

#### FN_vameGetEncoderAvailableChannels

获取可用编码通道数。

```c
VAME_ERR vameGetEncoderAvailableChannels(int* count);
```

#### FN_vameEncoderVideoCaps

获取视频编码能力。

```c
VAME_ERR vameEncoderVideoCaps(VAME_VideoEncCapability* caps);
```

## 数据结构类型

### 公共类型

#### VAME_Version

版本信息结构体。

```c
typedef struct {
    unsigned int major;
    unsigned int minor;
    unsigned int patch;
} VAME_Version;
```

#### VAME_Device

设备类型。

```c
typedef int VAME_Device;
```

#### VAME_ChId

通道 ID 类型。

```c
typedef int VAME_ChId;
```

#### VAME_Stream

流数据结构体。

```c
typedef struct {
    void* data;
    size_t size;
    long long pts;
    // ... 其他成员
} VAME_Stream;
```

#### VAME_Frame

帧数据结构体。

```c
typedef struct {
    void* data[3];      // Y/U/V 平面数据
    int stride[3];      // 每行字节数
    int width;
    int height;
    VAME_PixelFormat format;
    long long pts;
    // ... 其他成员
} VAME_Frame;
```

#### VAME_PixelFormat

像素格式枚举。

```c
typedef enum {
    VAME_PIXEL_FORMAT_NV12 = 0,
    VAME_PIXEL_FORMAT_NV21,
    VAME_PIXEL_FORMAT_YUV420P,
    VAME_PIXEL_FORMAT_RGB,
    VAME_PIXEL_FORMAT_BGR,
    // ... 其他格式
} VAME_PixelFormat;
```

#### VAME_CodecType

编解码器类型枚举。

```c
typedef enum {
    VAME_CODEC_JPEG = 0,
    VAME_CODEC_H264,
    VAME_CODEC_HEVC,
    // ... 其他类型
} VAME_CodecType;
```

### 解码相关类型

#### VAME_DecChannelParameters

解码通道参数结构体。

```c
typedef struct {
    VAME_CodecType codec_type;
    VAME_PixelFormat pixel_format;
    int width;
    int height;
    // ... 其他参数
} VAME_DecChannelParameters;
```

#### VAME_VideoInfo

视频信息结构体。

```c
typedef struct {
    int width;
    int height;
    VAME_CodecType codec;
    int fps_num;
    int fps_den;
    // ... 其他成员
} VAME_VideoInfo;
```

#### VAME_DecJpegInfo

JPEG 解码信息结构体。

```c
typedef struct {
    int width;
    int height;
    VAME_PixelFormat format;
    // ... 其他成员
} VAME_DecJpegInfo;
```

### 编码相关类型

#### VAME_EncChannelParameters

编码通道参数结构体。

```c
typedef struct {
    VAME_CodecType codec_type;
    int width;
    int height;
    int fps;
    int bitrate;
    VAME_EncRcMode rc_mode;
    // ... 其他参数
} VAME_EncChannelParameters;
```

#### VAME_EncVideoConfiguration

视频编码配置结构体。

```c
typedef struct {
    VAME_CodecType codec_type;
    int width;
    int height;
    int fps;
    int bitrate;
    VAME_EncRcMode rc_mode;
    VAME_EncQualityMode quality_mode;
    // ... 其他配置
} VAME_EncVideoConfiguration;
```

#### VAME_EncRcMode

码率控制模式枚举。

```c
typedef enum {
    VAME_RC_CBR = 0,    // 固定码率
    VAME_RC_VBR,        // 可变码率
    VAME_RC_CQP,        // 固定量化参数
    // ... 其他模式
} VAME_EncRcMode;
```

## 错误码定义

常见错误码：

```c
typedef enum {
    VAME_SUCCESS = 0,           // 成功
    VAME_ERR_INVALID_PARAM,     // 无效参数
    VAME_ERR_OUT_OF_MEMORY,     // 内存不足
    VAME_ERR_DEVICE_NOT_FOUND,  // 设备未找到
    VAME_ERR_TIMEOUT,           // 超时
    VAME_ERR_BUSY,              // 设备忙
    VAME_ERR_NOT_SUPPORTED,     // 不支持
    VAME_ERR_INVALID_STATE,     // 无效状态
    // ... 其他错误码
} VAME_ERR;
```

## 使用示例

### JPEG 同步解码

```c
#include <vame/vame.h>

// 1. 初始化
VAME_CommonFunctionList common_funcs;
vameCommonAPICreateInstance(&common_funcs);
common_funcs.vameInitialize(NULL);

// 2. 创建解码 API 实例
VAME_DecodeApiFunctionList decode_funcs;
vameDecodeAPICreateInstance(&decode_funcs);

// 3. 创建解码通道
VAME_ChId channel;
VAME_DecChannelParameters params = {
    .codec_type = VAME_CODEC_JPEG,
    .pixel_format = VAME_PIXEL_FORMAT_RGB,
};
decode_funcs.vameCreateDecoderChannel(&channel, &params);

// 4. 读取 JPEG 文件
FILE* fp = fopen("input.jpg", "rb");
fseek(fp, 0, SEEK_END);
size_t size = ftell(fp);
fseek(fp, 0, SEEK_SET);
void* jpeg_data = malloc(size);
fread(jpeg_data, 1, size, fp);
fclose(fp);

// 5. 同步解码
VAME_Frame frame;
decode_funcs.vameJpegSyncDecode(channel, jpeg_data, size, &frame);

// 6. 使用解码后的数据
printf("Decoded image: %dx%d\n", frame.width, frame.height);

// 7. 释放资源
decode_funcs.vameDecReleaseFrame(channel, &frame);
free(jpeg_data);
decode_funcs.vameDestroyDecoderChannel(channel);
common_funcs.vameUninitialize();
```

### H.264 视频解码

```c
// 1. 创建解码通道
VAME_ChId channel;
VAME_DecChannelParameters params = {
    .codec_type = VAME_CODEC_H264,
};
decode_funcs.vameCreateDecoderChannel(&channel, &params);

// 2. 启动解码器
decode_funcs.vameStartDecoder(channel);

// 3. 循环发送数据并接收帧
while (has_more_data) {
    VAME_Stream stream = {
        .data = video_packet,
        .size = packet_size,
        .pts = pts,
    };
    decode_funcs.vameSendStreamToDecoder(channel, &stream);
    
    // 接收解码后的帧
    VAME_Frame frame;
    while (decode_funcs.vameReceiveFrameFromDecoder(channel, &frame, 0) == VAME_SUCCESS) {
        // 处理帧
        process_frame(&frame);
        decode_funcs.vameDecReleaseFrame(channel, &frame);
    }
}

// 4. 刷新解码器
decode_funcs.vameSendStreamToDecoder(channel, NULL);

// 5. 接收剩余帧
VAME_Frame frame;
while (decode_funcs.vameReceiveFrameFromDecoder(channel, &frame, 1000) == VAME_SUCCESS) {
    process_frame(&frame);
    decode_funcs.vameDecReleaseFrame(channel, &frame);
}

// 6. 停止并销毁
decode_funcs.vameStopDecoder(channel);
decode_funcs.vameDestroyDecoderChannel(channel);
```

### H.264 视频编码

```c
// 1. 创建编码 API 实例
VAME_EncodeApiFunctionList encode_funcs;
vameEncodeAPICreateInstance(&encode_funcs);

// 2. 创建编码通道
VAME_ChId channel;
VAME_EncVideoConfiguration config = {
    .codec_type = VAME_CODEC_H264,
    .width = 1920,
    .height = 1080,
    .fps = 30,
    .bitrate = 4000000,
    .rc_mode = VAME_RC_VBR,
};
VAME_EncChannelParameters params = {
    .codec_type = VAME_CODEC_H264,
    .video_config = &config,
};
encode_funcs.vameCreateEncoderChannel(&channel, &params);

// 3. 启动编码器
encode_funcs.vameStartEncoder(channel);

// 4. 循环发送帧并接收编码数据
while (has_more_frames) {
    VAME_Frame frame = {
        .data = {y_data, u_data, v_data},
        .stride = {stride_y, stride_u, stride_v},
        .width = 1920,
        .height = 1080,
        .pts = pts,
    };
    encode_funcs.vameSendFrameToEncoder(channel, &frame);
    
    // 接收编码后的数据
    VAME_Stream stream;
    while (encode_funcs.vameReceiveStreamFromEncoder(channel, &stream, 0) == VAME_SUCCESS) {
        // 处理编码数据
        process_encoded_data(&stream);
        encode_funcs.vameEncReleaseStream(channel, &stream);
    }
}

// 5. 刷新编码器
encode_funcs.vameSendFrameToEncoder(channel, NULL);

// 6. 接收剩余数据
VAME_Stream stream;
while (encode_funcs.vameReceiveStreamFromEncoder(channel, &stream, 1000) == VAME_SUCCESS) {
    process_encoded_data(&stream);
    encode_funcs.vameEncReleaseStream(channel, &stream);
}

// 7. 停止并销毁
encode_funcs.vameStopEncoder(channel);
encode_funcs.vameDestroyEncoderChannel(channel);
```

## 注意事项

1. **实例管理**: 必须先创建实例获取函数指针才能调用相应接口
2. **资源释放**: 遵循"谁创建谁销毁"原则，确保所有资源被正确释放
3. **线程安全**: 同一通道不建议多线程同时访问
4. **错误处理**: 检查所有 API 返回值，正确处理错误
5. **性能优化**:
   - 复用通道，避免频繁创建销毁
   - 使用异步模式提高吞吐量
   - 合理设置编码参数

## 相关文档

- VastStream SDK 2.0 用户手册
- VAML API 参考手册
- VAME API 参考手册
