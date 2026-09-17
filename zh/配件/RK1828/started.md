# RK1828 AI 计算模块

## 一、产品介绍

### 产品简介

RK1828 AI 计算模块（型号：RM182XMC0）基于瑞芯微 RK1828 AI 协处理器打造，提供 20 TOPS INT8 AI 算力，片内集成 5GB 高带宽 3D 堆叠 DRAM，支持 3B~7B 参数量的 LLM/VLM 模型端侧推理。模块采用 M.2 2280（Key B-M）金手指设计，通过 PCIe 2.1 高速接口连接 RK3588、RK3576 等主控设备，即插即用，为主控设备灵活扩展 AI 算力。

| 正面 | 反面 |
| :---: | :---: |
| ![](../../../modules_img/RK1828/rk-1828-1.png) | ![](../../../modules_img/RK1828/rk-1828-2.png) |

### 详细参数

| 名称 | 参数 |
| --- | --- |
| 主控芯片 | RK1828（AI 协处理器） |
| AI 算力 | 20 TOPS（INT8） |
| 内存 | 5GB 片内 3D 堆叠 DRAM |
| CPU | 三核 RISC-V64 |
| 接口 | PCIe 2.1 |
| 外形规格 | M.2 2280（Key B-M） |
| 支持模型 | LLM（3B~7B 参数量）/ VLM / Omni / ASR / TTS / OCR / CV 等多类模型 |

## 二、使用方法

### 硬件安装

将 RK1828 AI 计算模块插入主控设备的 M.2 插槽，上电开机。主控通过 PCIe 高速接口连接协处理器:

* RK3588/RK3576 主控（Host）：作为系统核心，负责任务调度、资源分配和整体控制。
* RK1820/RK1828 协处理器（Device）：作为 AI 计算加速单元，专注于高性能神经网络推理任务。
* PCIe 高速接口（通信接口）：实现主控与协处理器之间的低延迟、高带宽数据交互。

### 编译 RK1820/RK1828 安装包

请联系销售 (sales@t-firefly.com) 获取 **RK182X SDK** 下载链接。

<font color=red>

**注意：**
<br>
**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**<br>
**2. 编译环境请使用 Ubuntu20.04或Ubuntu22.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**<br>
**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**<br>
**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**

</font>

<br>
比如，SDK 压缩包是 `rk182x_linux_release_20260908_v1.X.X.tgz`。（具体版本以网盘名称最新发布为准）

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf rk182x_linux_release_20260908_v1.X.X.tgz
.repo/repo/repo sync -l
```

#### Bundle 更新
1.1.0a 后续版本将以 bundle 的形式更新，以减少下载时间。
下载并解压上述 SDK 基础包、完成同步后，将 bundle 包放置在 SDK 根目录下：

```
rk182x_sdk/
├── .repo/
└── bundle_xx_to_xx.tgz
```

解压对应的 bundle 包：

```
tar xzf bundle_xx_to_xx.tgz
```

在 SDK 根目录运行 bundle 内的脚本：
```
./bundle_xx_to_xx/bundle_update.sh
```

#### 配置
通过 `./build.sh config` 配置。

```
Select board type:
1) RK182X EVB1
2) RK182X SODIMM
3) RK182X SODIMM USB
4) RK182X M2
5) Cancel
#?
```

选择 `4`


```
Select Security Boot Mode:
1) Disable Secure Boot
2) Enable Secure Boot
3) Cancel
#?
```

选择 `1`
> 该选项用于对固件进行安全加密，误操作可能导致固件后续无法升级，请谨慎选择。相关资料请参考1828SDK_Path/docs/Develop/Rockchip_RK1820_RK1828_User_Guide_SecureBoot_CN.pdf

#### 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_m2_installer_arm64.tgz`

### 安装软件包

手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_m2_installer_arm64.tgz` 到主控端
* 解压 `tar xzf rknn3_rk182x_m2_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后，RK3588 或者 RK3576 端系统会在启动后，自动下载 RK182X 的固件，并启动后台服务程序。


### 编译deb
firefly 提供的sdk中，在1828sdk_path/tools/build_helper补充了对应的deb包脚本，可以直接运行/tools/build_helper/build-helper.sh sodimm m2
会生成对应的deb包到目录下，这样就能够直接dpkg -i xxx.deb进行安装了。
> Rockchip原生提供的deb编译会安装源代码并需求网络环境，安装编译工具并编译驱动。由于我们的固件里面已经编译了对应的rkep驱动，因此提供的sdk中已调整deb编译脚本，不再装入驱动代码并检测编译工具。

### rknn-smi

rknn-smi (System Management Interface) 工具用于 RK1820/RK1828 设备信息收集、功能配置、日志管理等功能。

* 查询软件版本信息: `sudo rknn-smi -v`
* 查询硬件版本信息: `sudo rknn-smi info -l`
* 状态监控: `sudo rknn-smi info -w`
* 性能模式: `sudo rknn-smi set -t work_mode -s 2`
* 查询功耗：此功能硬件上不支持
    * `sudo rknn-smi info -t power`

`sudo rknn-smi -v` 版本信息参考（V 1.1.0）：

```
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.1
RC chips connect version      : 3.3.2
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.1.0
rknn3 API version             : 1.1.0
```

具体使用，详见 RK182X SDK 里的 `docs/Tools/Rockchip_User_Guide_RKNN-SMI_Tool_CN.pdf`

### RKNN3

RK182X SDK 的 rknn 目录：
```
rknn/
├── rknn3-model-zoo
├── rknn3-runtime
├── rknn3-toolkit
└── rknn-gstreamer-plugins
```

**RKNN3 SDK 框图**
<center>

![](../../../modules_img/RK1828/rknn3-sdk-block-diagram.png)
</center>

典型工作流程：先在 PC 上使用 RKNN3-Toolkit 将训练好的模型转换为 RKNN 格式，再通过 RKNN3 Runtime API 在开发板上进行推理。

#### RKNN3 Model Zoo
提供 RK1820/RK1828 平台上经典模型的部署示例。更多详细内容可以参考[github](https://github.com/airockchip/rknn3-model-zoo)

#### RKNN3 Runtime
RKNN3 C API 是 RKNN3 Runtime（运行时库）的 C 语言接口。开发者使用 C/C++ 开发应用程序，通过 RKNN3 C API 部署模型推理。

#### RKNN3 Toolkit
RKNN3 Toolkit 是为用户提供在 PC 平台上进行模型转换、推理和性能评估的开发套件。支持的 Python 版本：Python 3.10、Python 3.12。

**RKNN3 Toolkit** 与 [RKNN-Toolkit](https://github.com/airockchip/rknn-toolkit) 和 [RKNN-Toolkit2](https://github.com/airockchip/rknn-toolkit2) **不兼容**。更多内容可以参考[github](https://github.com/airockchip/rknn3-toolkit)

#### 预转换 RKNN 模型
用户可以从 [RKNN3_SDK 网盘](https://console.box.lenovo.com/l/H1fig1) 下载预先转换好的 RKNN 模型（提取码：`rknn`），无需自行转换。当前版本（V1.1.0）发布的模型位于 `RKNN3_SDK/rknn3_models/v1.1.0` 目录。

### 常见问题

#### 概率性 "Failed to initialize rknnsmi"
`/lib/systemd/system/rknn3.service` 适当添加延迟：

```
[Unit]
Description=rknn3 runtime service
DefaultDependencies=no
After=local-fs.target

[Service]
Type=forking
ExecStartPre=/bin/sleep 3 # 延迟 3 秒
ExecStart=/bin/rknn3_startup start
ExecStop=/bin/rknn3_startup stop

[Install]
WantedBy=sysinit.target
```

#### 当前支持的模型

当前 SDK（V1.1.0）支持的模型类别包括 LLM、VLM、Omni、ASR（语音识别）、TTS（文本转语音）、Embedding / Reranker、翻译、OCR、CV（计算机视觉）等，例如 Qwen3 / Qwen3.5 / GLM-Edge / MiniCPM5（LLM），Qwen3-VL / InternVL3.5 / SmolVLM2（VLM），Whisper / SenseVoice（ASR），PaddleOCR VL（OCR），YOLOv8 / YOLO26（CV）等。各模型的完整列表和详细说明请参考 SDK 中的 `rknn/rknn3-runtime/doc/CN/00_RKNN3_SDK_发布说明_V1.1.0.pdf`。

#### rknn3 API Version 显示 NA
安装一下binutils, 部分rootfs可能没有strings指令

```sh
sudo apt update
sudo apt install binutils
```

## 三、更多资料

* RKNN3 Model Zoo: https://github.com/airockchip/rknn3-model-zoo
* RKNN3 Toolkit: https://github.com/airockchip/rknn3-toolkit

