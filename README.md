# 瀚博半导体 Vastai AI Skills

> ⚠️ **非官方项目 / Unofficial Project**
> 本仓库为社区整理，与瀚博半导体官方无关。
> This repository is community-maintained and **not affiliated with Vastai Technology Co., Ltd.**

---

## 📖 简介

本仓库收录了一组专为 AI 软件开发工具（如 Cursor、Windsurf、GitHub Copilot、Cline 等）设计的 **Agent Skills**，覆盖瀚博半导体（Vastai）AI 加速卡的完整开发工具链，包括模型转换、推理部署、视频编解码、硬件监控等核心场景。

每个 Skill 包含：
- ✅ 详细的 API 参考与使用说明（`SKILL.md`）
- ✅ 典型使用场景与示例
- ✅ 最佳实践与常见问题

---

## 📦 包含内容

共收录 **15 个 Skills**，涵盖以下分类：

### 🚀 部署与环境
| Skill 目录 | 说明 |
|---|---|
| `vastai-developer-center` | 开发者中心部署指南，支持 DeepSeek-V3/R1/V3.1、Qwen3 系列模型（vLLM+VACC） |
| `vastai-docker` | Vastai Docker 安装与配置，用于容器化 AI 工作负载 |
| `vastai-k8s-device-plugin` | Kubernetes 设备插件，在 K8s 集群中管理 Vastai AI 加速卡 |

### 🔧 模型工具链
| Skill 目录 | 说明 |
|---|---|
| `vastai-vamc` | VAMC 3.4.3 模型转换工具，将深度学习模型转换为 Vastai 格式 |
| `vastai-vamp` | VAMP 模型性能分析工具，测试吞吐量、延迟、功耗和精度 |

### 📹 视频编解码 SDK
| Skill 目录 | 说明 |
|---|---|
| `vastai-vame-api` | VAME API 2.8.3，基于 C API 开发视频/图像编解码应用 |
| `vastai-vaststream-sdk-cpp` | VastStream SDK 2.0 C++ 用户手册，深度学习推理与视频编解码 |
| `vastai-vaststream-sdk-python` | VastStream SDK 2.0 Python 用户手册 |
| `vastai-vaststreamx-cpp` | VastStreamX C++ API 手册，模型部署与视频处理 |
| `vastai-vaststreamx-python` | VastStreamX 2.8.16 Python 手册，深度学习推理应用开发 |

### 📊 监控与运维
| Skill 目录 | 说明 |
|---|---|
| `vastai-vaml-api` | VAML（Vastai Management Library）API，硬件监控与设备状态检测 |
| `vastai-vasmi` | Vasmi 硬件信息监测与管理工具，查询设备状态、管理进程 |
| `vastai-vaprofiler` | Vaprofiler 芯片监测分析，视频编解码吞吐量与 AI 利用率监控 |
| `vastai-valogger` | Valogger 芯片日志导出工具，导出与分析 SV100 系列固件运行日志 |
| `vastai-exporter` | Vastai Exporter，为 Prometheus 提供 AI 加速卡监控指标 |

---

## 🚀 快速开始

### 获取 Skills

克隆本仓库，或将所需 Skill 目录复制到你的 AI 工具对应的 skills 路径下：

```bash
git clone https://github.com/CreamyLong/vastskills.git
```

各工具 skills 目录参考：

| 工具 | Skills 目录 |
|---|---|
| Windsurf | `~/.windsurf/skills/` |
| Cursor | `~/.cursor/skills/` |
| Cline | 参考工具文档 |

或直接下载单个 Skill：

```bash
# 仅下载 VastStreamX Python skill
mkdir -p vastai-vaststreamx-python
curl -o vastai-vaststreamx-python/SKILL.md \
  https://raw.githubusercontent.com/CreamyLong/vastskills/main/vastai-vaststreamx-python/SKILL.md
```

### 使用示例

配置好 Skill 后，直接向 AI 助手提问：

```
# 模型部署
如何在 Vastai 硬件上用 vLLM 部署 DeepSeek-V3？

# 模型转换
如何用 VAMC 将 ONNX 模型转换为 Vastai 格式？

# 视频推理
如何用 VastStreamX Python SDK 实现目标检测推理？

# 硬件监控
如何用 vasmi 查询 Vastai 加速卡的温度和利用率？
```

---

## 📋 适用硬件

- 瀚博半导体 SV100 系列 AI 加速卡
- 支持 VACC（Vastai AI Compute Cluster）架构

---

## 🤝 贡献

欢迎提交 PR 改进已有 Skill 文档，或补充新的使用场景！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b improve/vastai-vamc`)
3. 提交修改 (`git commit -m 'Improve VAMC skill examples'`)
4. 推送分支 (`git push origin improve/vastai-vamc`)
5. 发起 Pull Request

---

## ⚠️ 免责声明

- 本仓库为**非官方**社区项目，内容来源于公开文档整理
- 瀚博半导体官方信息请访问 [www.vastai.com](https://www.vastai.com)
- SDK 版本以官方发布为准，本仓库内容可能存在滞后

---

## 📄 许可证

MIT License — 详见 [LICENSE](LICENSE)
