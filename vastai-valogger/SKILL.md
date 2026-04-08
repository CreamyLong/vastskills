---
name: "vastai-valogger"
description: "瀚博半导体 Valogger 芯片日志导出工具。Invoke when user needs to export, parse, or analyze SV100 series chip firmware runtime logs on Vastai accelerators."
---

# Valogger 工具使用指南

Valogger 是芯片日志导出工具，可将 SV100 系列芯片固件运行时产生的日志导出，并以文件形式进行保存。用户可以使用 Valogger 了解芯片上的固件运行的状态与运行流程。

## 功能特性

- 导出 SV100 系列芯片固件运行时日志
- 按天存储日志文件
- 支持多种日志文件格式（txt、bin）
- 支持按设备、Die、模块过滤日志
- 支持热插拔场景
- 支持设置日志读取时间间隔

## 日志存储特点

- 按天存储
- 每个 core 存储的日志最大为 200 MB
- 每个文件最大为 20 MB
- 最多保存近 7 天的日志文件
- 日志文件默认存储在 `/var/log/valog/fwlog` 路径下

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
valogger --version
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
valogger [options]
```

### 通用命令

#### 查询帮助

```bash
valogger --help
```

输出信息：
```
--help              print help information
-d                  grabs the specified device log
-i                  grabs the specified die log
--version           print version information
--filetype          specifies the file type to dump and parse the DDR log
```

#### 查询版本

```bash
valogger --version
```

#### 导出固件运行日志到文件夹

```bash
valogger -d {device_id} -i {die_id}
```

示例：
```bash
# 导出设备 0 的 Die 0 日志
valogger -d 0 -i 0

# 导出设备 1 的所有日志
valogger -d 1
```

#### 设置日志导出文件格式

```bash
valogger --filetype {type}
```

文件类型说明：
- 0: txt 格式
- 1: bin 格式
- 2: 默认格式

示例：
```bash
valogger --filetype 0
```

#### 解析单个文件

```bash
valogger --parse {filepath}
```

#### 导出特定模块日志到文件

```bash
valogger --module {module_name} -o {output_file}
```

#### 导出日志，指定日志文件夹为 SN

```bash
valogger --sn
```

#### 热插拔支持

```bash
valogger --hotplug
```

注意：热插拔模式仅适用于 VA1M 卡。

#### 指定日志读取时间间隔

```bash
valogger --interval {milliseconds}
```

示例：
```bash
# 设置读取间隔为 1000 毫秒（1 秒）
valogger --interval 1000
```

## 配置文件

### 配置格式

配置文件使用 YAML 格式：

```yaml
# valogger 配置文件示例
log:
  path: /var/log/valog/fwlog
  max_size: 20MB
  max_age: 7d
  max_backups: 7

filter:
  devices: [0, 1, 2]
  dies: [0, 1]
  modules: ["core", "memory", "pcie"]

output:
  format: txt
  timestamp: true
```

### 模板样式

标准配置文件模板：

```yaml
# Valogger Configuration File
# 日志存储配置
storage:
  base_path: /var/log/valog/fwlog
  max_file_size: 20MB
  max_core_size: 200MB
  retention_days: 7

# 日志过滤配置
filter:
  # 指定设备 ID 列表，为空表示所有设备
  device_ids: []
  # 指定 Die ID 列表，为空表示所有 Die
  die_ids: []
  # 指定模块列表，为空表示所有模块
  modules: []

# 输出配置
output:
  # 输出格式: txt, bin
  format: txt
  # 是否包含时间戳
  include_timestamp: true
  # 是否按 SN 组织文件夹
  organize_by_sn: false
```

## 日志文件

### 存放路径

默认路径：`/var/log/valog/fwlog`

### 存放规则

```
/var/log/valog/fwlog/
├── 2024-01-01/
│   ├── device_0_die_0_core_0.log
│   ├── device_0_die_0_core_1.log
│   ├── device_0_die_1_core_0.log
│   └── ...
├── 2024-01-02/
│   └── ...
└── latest -> 2024-01-02/
```

## 使用示例

### 示例 1：导出所有设备日志

```bash
valogger
```

### 示例 2：导出指定设备日志

```bash
valogger -d 0
```

### 示例 3：导出指定 Die 日志

```bash
valogger -d 0 -i 0
```

### 示例 4：使用配置文件导出

```bash
valogger -c /path/to/config.yaml
```

### 示例 5：导出为 bin 格式

```bash
valogger --filetype 1 -d 0
```

### 示例 6：热插拔场景导出

```bash
valogger -d 0 --hotplug
```

## 注意事项

1. 在 Docker 容器内和宿主机上并行运行 Valogger 会导致采集的日志内容不符合预期
2. 建议仅在 Docker 容器内或仅在宿主机上使用 Valogger，不要同时在两者上运行
3. 确保有足够的磁盘空间存储日志文件
4. 定期清理过期日志以释放空间
5. 设置合适的时间间隔以避免过度占用系统资源

## 故障排查

| 问题 | 解决方案 |
|------|----------|
| 无法导出日志 | 检查是否有足够的磁盘空间 |
| 日志文件为空 | 检查设备是否正常运行 |
| 权限错误 | 使用 sudo 或以 root 用户运行 |
| 热插拔失败 | 确认是否为 VA1M 卡 |
| 日志解析错误 | 检查日志文件格式是否正确 |

## 日志分析

### 常见日志字段

| 字段 | 说明 |
|------|------|
| timestamp | 时间戳 |
| device_id | 设备 ID |
| die_id | Die ID |
| core_id | Core ID |
| module | 模块名称 |
| level | 日志级别（DEBUG/INFO/WARN/ERROR） |
| message | 日志内容 |

### 日志级别说明

- **DEBUG**: 调试信息
- **INFO**: 一般信息
- **WARN**: 警告信息
- **ERROR**: 错误信息
