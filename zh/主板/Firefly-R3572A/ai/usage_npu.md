# NPU 使用

## 简介 [step]

Firefly-R3572A 搭载的 RK3572 内置 NPU，算力 4 TOPS（稀疏 INT8），支持 INT4/INT8/INT16/FP4/FP8/FP16/BF16 混合运算与 W4A16 非对称 MAC 运算，可用于 CNN、YOLO 目标检测以及大语言模型等 AI 应用的边缘侧推理。

<Callout title="重要" type="warn">

RK3572 的 NPU 使用全新的 **RKNN3 SDK** 软件栈，**与 RKNN-Toolkit2 / RKNPU2 不兼容**。RK3588/RK3576/RK3568 等平台使用的 `rknn-toolkit2`、`librknnrt.so`、`rknn_server`（即 RKNPU2 体系）**不适用于 RK3572**。请使用本章介绍的 RKNN3-Toolkit 与 RKNN3-Runtime。

</Callout>

## RKNN3 SDK 介绍 [step]

RKNN3 SDK 提供将 AI 模型部署到 RK1820/RK1828/**RK3572** 的完整软件栈，包含以下组件：

* **RKNN3-Toolkit**：PC 端开发套件，支持模型转换、量化、推理和性能评估
* **RKNN3-Runtime**：板端运行时库（`librknn3_api.so`），提供 C/C++ 编程接口，用于加载 RKNN 模型并在 NPU 上推理
* **RKNN3 Model Zoo**：CNN / LLM / VLM 等模型的转换与部署示例

各组件信息如下：

| 组件 | 说明 | 位置 |
| --- | --- | --- |
| RKNN3-Toolkit | PC 端模型转换工具（Python 3.10 / 3.12） | SDK `external/rknn3-toolkit/`，开源地址 [rknn3-toolkit](https://github.com/airockchip/rknn3-toolkit) |
| RKNN3-Runtime | 板端运行库与头文件 `rknn3_api.h` | SDK `external/rknn3-runtime/rknn3-api/` |
| RKNN3 Model Zoo | 模型转换与部署示例 | [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo) |

当前 SDK 版本为 **RKNN3 SDK V1.1.0**，板端 Runtime 构建包版本 `RKNN3_RUNTIME_VERSION = 1.0.0`。详细支持平台与版本说明以 SDK 内 `external/rknn3-runtime/README_CN.md` 与发布说明为准。

## NPU 驱动与设备节点 [step]

Firefly-R3572A 的 RK3572 使用 **RKNPU3 内核驱动**（设备节点 `/dev/rknpu3`，sysfs 目录 `/sys/class/misc/rknpu3`），对应内核配置 `CONFIG_ROCKCHIP_RKNPU3=y`。查看驱动版本与硬件信息请从内核日志获取：

```bash
# 查看 RKNPU3 驱动版本（如 RKNPU3 Driver v1.0.3 (20260613)）
dmesg | grep -i RKNPU3

# 查看 NPU 设备节点与 sysfs 目录
ls -l /dev/rknpu3
ls /sys/class/misc/rknpu3/

# 查看 NPU 当前频率
cat /sys/class/misc/rknpu3/device/devfreq/*/cur_freq
```

<Callout title="注意" type="info">

网上常见的 `cat /sys/kernel/debug/rknpu/version` 属于旧版 **rknpu** 驱动的 debugfs 接口，RK3572 实际绑定的是 RKNPU3 驱动（`/sys/class/misc/rknpu3`、`/dev/rknpu3`），并无该 debugfs 路径，请用上面的 `dmesg` 方式查看驱动版本。

</Callout>

## 板端 Runtime 环境 [step]

RKNN3-Runtime 提供板端 C/C++ API，核心头文件为 `rknn3_api.h`，库文件为 `librknn3_api.so`，按目标架构位于 `external/rknn3-runtime/rknn3-api/` 下（如 `Linux/aarch64/`、`Android/arm64-v8a/`）。

* 头文件：`rknn3-api/include/rknn3_api.h`
* 动态库：`rknn3-api/Linux/aarch64/librknn3_api.so`（Linux）、`rknn3-api/Android/arm64-v8a/librknn3_api.so`（Android）

使用时将头文件路径与库文件路径加入工程，链接 `librknn3_api.so` 即可调用 RKNN3 Runtime API。板端构建与部署示例可参考 SDK `external/rknn3-test/`（`build-linux.sh`、`build-android.sh` 及 `rknn3_stress_test/` 下的 CNN / LLM 测试程序）。

<Callout title="注意" type="warn">

RKNN3-Toolkit、RKNN3-Runtime 与固件中的 NPU 驱动需要版本匹配。RK3572 当前固件配套 RKNPU3 驱动 v1.0.3 与 RKNN3 SDK V1.1.0。请勿混用旧版 RKNN-Toolkit2 / RKNPU2 体系的库或模型，否则会出现模型无法加载或推理异常。

</Callout>

## 模型转换 [step]

在 PC（Ubuntu，Python 3.10 或 3.12）上安装 RKNN3-Toolkit，将训练好的模型转换为 RKNN 格式。具体安装方式、API 与转换参数请参考 SDK `external/rknn3-toolkit/` 内的文档（CN/EN）以及 [rknn3-toolkit](https://github.com/airockchip/rknn3-toolkit)。

转换及部署示例可参考 RKNN3 Model Zoo（[rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo)），其中提供了 CNN、LLM、VLM 等模型的完整转换与板端部署例程。SDK 网盘也提供了预转换好的 RKNN 模型（位于 `RKNN3_SDK/rknn3_models/v1.1.0`）。

## 板端推理 [step]

将转换后的 `.rknn` 模型与 RKNN3-Runtime 库部署到主板后，通过 `rknn3_api.h` 提供的接口加载模型、输入数据并获取推理结果。SDK `external/rknn3-test/rknn3_stress_test/` 下提供了可直接编译运行的 CNN 测试示例，可作为接入参考。

大语言模型（LLM）的部署使用 RKLLM3，详见 [大语言模型](usage_rkllm.md)。

## 验证部署 [step]

运行 SDK 提供的 RKNN3 示例程序（如 `external/rknn3-test/` 编译产物的 CNN 测试），确认模型加载与推理正常。若出现初始化失败，请依次检查：

1. `/dev/rknpu3` 是否存在、RKNPU3 驱动是否正常加载（`dmesg | grep -i RKNPU3`）；
2. 模型是否为 RKNN3-Toolkit 转换、与板端 Runtime 版本是否匹配；
3. 是否误用了 RKNPU2 / RKNN-Toolkit2 体系的库或模型。

## 详细开发文档 [step]

RK3572 NPU 及 RKNN3 SDK 详细使用方法请参考 SDK `external/rknn3-runtime/doc/`、`external/rknn3-toolkit/doc/` 下的文档，以及 [RKNN3-Toolkit](https://github.com/airockchip/rknn3-toolkit)、[RKNN3-Model-Zoo](https://github.com/airockchip/rknn3-model-zoo) 开源仓库。