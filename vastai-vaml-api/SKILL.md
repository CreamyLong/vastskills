---
name: "vastai-vaml-api"
description: "瀚博半导体 VAML (Vastai Management Library) API 参考手册。Invoke when user needs to use VAML library for hardware monitoring, device information retrieval, or card status detection on Vastai AI accelerators."
---

# VAML 1.0 API 参考手册

VAML（Vastai Management Library）为第三方应用提供硬件监测与板卡状态检测接口管理平台，是 vasmi 和 vaprofiler 工具的界面显示底层支持库。

## 功能特性

- 获取设备信息
- 监测 MPU 使用情况
- 检测板卡状态
- 支持多线程编程

## 数据结构

### vamlDeviceHandle

```c
typedef int vamlDeviceHandle
```

用于获取卡信息的结构体。

### vamlDieHandle

```c
typedef int vamlDieHandle
```

用于获取 Die 信息的结构体。

### vamlDieMemory_struct

```c
struct vamlDieMemory_struct {
    unsigned long long total;  // 总内存
    unsigned long long free;   // 空闲内存
    unsigned long long used;   // 使用内存
};
```

用于获取内存信息的结构体。

### vamlDieUtilizeTotal_struct

```c
struct vamlDieUtilizeTotal_struct {
    int vdsp;   // vdsp 利用率汇总（万分比）
    int vemcu;  // vemcu 利用率汇总（万分比）
    int vdmcu;  // vdmcu 利用率汇总（万分比）
};
```

用于获取 MCU 使用情况汇总的结构体。

### vamlDieUtilize_struct

```c
struct vamlDieUtilize_struct {
    unsigned int vdsp;   // vdsp 利用率（万分比）
    unsigned int vemcu;  // vemcu 利用率（万分比）
    unsigned int vdmcu;  // vdmcu 利用率（万分比）
};
```

用于获取设备利用率明细的结构体，共包含 4 个 vdsp、4 个 vemcu、3 个 vdmcu。

## API 参考

### vamlInit

```c
vamlOpStatus_t vamlInit(void);
```

**功能描述**: VAML 库初始化，完成系统设备和库所需内存资源的申请等工作。

**调用时机**: 在使用库最开始阶段调用一次，后期调用将不再初始化。

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 库在设备信息获取或者实例化过程中失败

### vamlShutDown

```c
vamlOpStatus_t vamlShutDown(void);
```

**功能描述**: VAML 库释放关闭，用于释放设备资源和初始化过程中申请的资源信息。

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlLogLevelSet

```c
vamlOpStatus_t vamlLogLevelSet(int level);
```

**功能描述**: 设置日志级别。

**参数**:
- level: 日志级别

### vamlGetVersion

```c
vamlOpStatus_t vamlGetVersion(char *version);
```

**功能描述**: 获取 VAML 库版本信息。

**参数**:
- version: 版本字符串缓冲区

### vamlErrorString

```c
const char* vamlErrorString(vamlOpStatus_t error);
```

**功能描述**: 将错误码转换为错误描述字符串。

**参数**:
- error: 错误码

**返回值**: 错误描述字符串

### vamlDeviceGetCount

```c
vamlOpStatus_t vamlDeviceGetCount(unsigned int *count);
```

**功能描述**: 获取系统中设备数量。

**参数**:
- count: 返回设备数量

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDeviceGetHandleByIndex

```c
vamlOpStatus_t vamlDeviceGetHandleByIndex(unsigned int index, vamlDeviceHandle *device);
```

**功能描述**: 通过索引获取设备句柄。

**参数**:
- index: 设备索引（从 0 开始）
- device: 返回设备句柄

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDeviceGetDieCount

```c
vamlOpStatus_t vamlDeviceGetDieCount(vamlDeviceHandle device, unsigned int *count);
```

**功能描述**: 获取指定设备的 Die 数量。

**参数**:
- device: 设备句柄
- count: 返回 Die 数量

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDieGetHandleByIndex

```c
vamlOpStatus_t vamlDieGetHandleByIndex(vamlDeviceHandle device, unsigned int index, vamlDieHandle *die);
```

**功能描述**: 通过索引获取 Die 句柄。

**参数**:
- device: 设备句柄
- index: Die 索引
- die: 返回 Die 句柄

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDeviceGetDieMemory

```c
vamlOpStatus_t vamlDeviceGetDieMemory(vamlDieHandle die, vamlDieMemory_struct *memory);
```

**功能描述**: 获取指定 Die 的内存信息。

**参数**:
- die: Die 句柄
- memory: 返回内存信息结构体

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDeviceGetDieUtilize

```c
vamlOpStatus_t vamlDeviceGetDieUtilize(vamlDieHandle die, vamlDieUtilize_struct *utilize);
```

**功能描述**: 获取指定 Die 的利用率明细。

**参数**:
- die: Die 句柄
- utilize: 返回利用率结构体

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

### vamlDeviceGetDieUtilizeTotal

```c
vamlOpStatus_t vamlDeviceGetDieUtilizeTotal(vamlDieHandle die, vamlDieUtilizeTotal_struct *total);
```

**功能描述**: 获取指定 Die 的利用率汇总。

**参数**:
- die: Die 句柄
- total: 返回利用率汇总结构体

**返回值**:
- VAML_SUCCESS: 成功
- 其他值: 失败

## 错误码定义

| 错误码 | 值 | 说明 |
|--------|-----|------|
| VAML_SUCCESS | 0 | 成功 |
| VAML_ERROR_INVALID_ARGUMENT | 1 | 无效参数 |
| VAML_ERROR_NOT_INITIALIZED | 2 | 库未初始化 |
| VAML_ERROR_NO_PERMISSION | 3 | 无权限 |
| VAML_ERROR_NOT_FOUND | 4 | 设备未找到 |
| VAML_ERROR_UNKNOWN | 5 | 未知错误 |

## 使用示例

```c
#include <stdio.h>
#include "vaml.h"

int main() {
    vamlOpStatus_t status;
    unsigned int deviceCount;
    vamlDeviceHandle device;
    vamlDieHandle die;
    vamlDieMemory_struct memory;
    vamlDieUtilizeTotal_struct utilize;
    
    // 初始化 VAML 库
    status = vamlInit();
    if (status != VAML_SUCCESS) {
        printf("VAML init failed: %s\n", vamlErrorString(status));
        return -1;
    }
    
    // 获取设备数量
    status = vamlDeviceGetCount(&deviceCount);
    if (status != VAML_SUCCESS) {
        printf("Get device count failed: %s\n", vamlErrorString(status));
        vamlShutDown();
        return -1;
    }
    printf("Device count: %u\n", deviceCount);
    
    // 获取第一个设备句柄
    status = vamlDeviceGetHandleByIndex(0, &device);
    if (status != VAML_SUCCESS) {
        printf("Get device handle failed: %s\n", vamlErrorString(status));
        vamlShutDown();
        return -1;
    }
    
    // 获取第一个 Die 句柄
    status = vamlDieGetHandleByIndex(device, 0, &die);
    if (status != VAML_SUCCESS) {
        printf("Get die handle failed: %s\n", vamlErrorString(status));
        vamlShutDown();
        return -1;
    }
    
    // 获取内存信息
    status = vamlDeviceGetDieMemory(die, &memory);
    if (status == VAML_SUCCESS) {
        printf("Memory - Total: %llu, Free: %llu, Used: %llu\n",
               memory.total, memory.free, memory.used);
    }
    
    // 获取利用率汇总
    status = vamlDeviceGetDieUtilizeTotal(die, &utilize);
    if (status == VAML_SUCCESS) {
        printf("Utilization - VDSP: %d, VEMCU: %d, VDMCU: %d (per 10000)\n",
               utilize.vdsp, utilize.vemcu, utilize.vdmcu);
    }
    
    // 关闭 VAML 库
    vamlShutDown();
    return 0;
}
```

## 编译链接

```bash
gcc -o vaml_example vaml_example.c -lvaml
```

## 注意事项

1. 必须在程序开始时调用 vamlInit() 进行初始化
2. 程序结束前应调用 vamlShutDown() 释放资源
3. 支持多线程编程，多个线程可以同时调用 VAML 接口
4. 所有 API 都返回 vamlOpStatus_t 类型的状态码
5. 利用率的单位为万分比（即 10000 表示 100%）
