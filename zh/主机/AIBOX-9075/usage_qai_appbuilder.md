# QAI AppBuilder

## 工具介绍

QAI AppBuilder 是运行在 Qualcomm 设备上的 AI agent，可以帮助用户在设备上部署各类 AI 应用。接入大模型后，用户可以使用自然语言指挥 agent 完成开发、配置和部署任务，降低 AI 应用落地门槛，极大方便 AI 应用的快速落地。

QAI AppBuilder 的使用需要注册并登录 Qualcomm 账号，这是使用该工具的必要账号。首次注册 Qualcomm 账号会赠送 10M 免费 token，可用于体验大模型服务。

用户还可以接入其他大模型供应商，根据需求选择合适的云端模型。

如果要使用 QAI AppBuilder 中的 AI Hub 功能，则还需要另外注册 AI Hub 账号并获取 API key。

### 功能模块

根据官方项目文档，QAI AppBuilder 的主要功能模块包括以下内容。不同版本和不同 QCOM 设备的可用模型、功能入口可能有所差异。

| 功能模块 | 主要能力 |
| --- | --- |
| **App Builder（应用构建器）** | 在设备上运行打包好的 Model Pack。用户可以上传图片、录音或文本，选择模型、调整参数并执行推理，适合快速搭建图像、语音等端侧 AI 应用。 |
| **Model Builder（模型构建器）** | 通过自然语言指导模型转换、量化、推理和验证，将 PyTorch 或 ONNX 模型转换为适合 Qualcomm NPU 使用的 QNN 格式。该功能主要面向中小型模型，官方建议模型规模约为 2 GB 以下，暂不用于转换 LLM。 |
| **AI Hub（模型目录与下载中心）** | 浏览和下载经过 Qualcomm NPU 优化的模型及相关资源，支持断点续传、校验和等能力，减少手动准备模型的工作量。 |
| **多 Agent 流式聊天** | 支持实时流式对话、Markdown 和代码高亮，并可使用讨论模式，让多个 Agent 围绕一个主题协作讨论并形成实施方案。 |
| **LLM Agent Pipeline** | 云端大模型可以通过 `appbuilder_run` 和 `appbuilder_batch_run` 等工具，自主编排设备上的多个 Model Pack，完成“筛选图片、执行检测、再进行处理”等复合任务。 |
| **Skill 扩展系统** | 通过内置或用户自定义 Skill 扩展 Agent 能力，支持热加载和将 Skill 提示词自动注入对话。 |
| **设备服务与模型运行管理** | 可管理本地 NPU 推理服务，包括服务启动、停止、状态查看、模型加载和运行日志查看。 |

### 核心亮点

- **自然语言驱动开发**：用户无需先掌握完整的 Qualcomm AI 工具链，只需描述目标，Agent 就可以协助完成模型选择、参数配置、应用生成和部署验证。
- **Model Pack 自包含应用形态**：一个 Model Pack 可以包含模型清单、运行脚本、参数 Schema 和可选 Skill，便于复制、管理和扩展。
- **本地 NPU 推理与隐私保护**：应用推理在 QCOM 设备本地完成，输入数据不会因为本地推理而上传；如果使用云端大模型，云端主要处理用户主动发送的文本或结果摘要。
- **多精度模型切换**：同一个模型可以提供 FP16、INT8、W8A16 等多个精度变体，用户可按精度、速度、内存和模型大小选择合适的版本。
- **Sticky Worker 加速重复推理**：模型首次运行完成加载和预热后，后续运行可以复用已加载的上下文，减少重复初始化开销。
- **历史、对比与基准测试**：保存每次运行记录，支持历史回看、多次结果对比、批量运行以及 p50、p90、p99 等延迟统计。
- **Agent 自动编排多个模型**：在连接云端大模型后，可以用一句自然语言描述复杂任务，由 Agent 自动拆解步骤并串联多个本地模型完成任务。
- **安全的权限控制**：文件访问、命令执行和外部路径访问可以经过权限审批；API key 等敏感信息应通过应用提供的安全配置方式保存，避免明文写入文档或脚本。

> 项目及下载地址: [点击前往](https://github.com/qualcomm/qai-appbuilder/tree/main)  
> Qualcomm AI Hub: [点击前往](https://aihub.qualcomm.com/)

## 安装 QAI AppBuilder

以下步骤适用于运行 Ubuntu 的 QCOM 设备。

> **网络提示：** 中国大陆用户使用 QAI AppBuilder 前，需要提前为系统设置可访问相关服务的网络代理。

### 1. 安装 nvm

请按照 [nvm 官方项目](https://github.com/nvm-sh/nvm) 的说明安装 nvm。例如：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

安装完成后重新打开终端，确认 `nvm` 命令可用。

### 2. 安装 Node.js 22 和 pnpm

```bash
nvm install 22
npm install -g pnpm
```

### 3. 安装系统依赖

```bash
sudo apt install cmake build-essential python3.12-dev python3.12-venv
```

### 4. 安装 QAI AppBuilder

将 `qaiappbuilder.zip` 复制到设备后解压，并执行压缩包中的 `setup.sh`：

```bash
unzip qaiappbuilder.zip
cd qaiappbuilder
chmod +x setup.sh start.sh
./setup.sh
```

如果解压后目录名称不同，请进入实际包含 `setup.sh` 的目录再执行命令。

## 启动与使用

安装完成后执行：

```bash
./start.sh
```

服务启动后，在浏览器打开 [http://localhost:8989](http://localhost:8989) 开始聊天。首次使用时，根据页面提示登录 Qualcomm 账号。如果需要使用 AI Hub 功能，再注册 AI Hub 账号并配置对应的 API key；也可以在设置中接入其他兼容的大模型供应商。

通过自然语言描述目标后，QAI AppBuilder agent 可以协助完成模型选择、应用开发和部署。具体功能与版本说明请参考 [QAI AppBuilder 项目文档](https://github.com/qualcomm/qai-appbuilder/blob/main/tools/qaiappbuilder/README.zh-CN.md)。

## 使用示例

下面以一个递进式流程为例，展示如何使用 QAI AppBuilder 完成 AI 应用开发：先询问并确定合适的模型，再制作图片检测应用，最后基于前一步的模型和应用结果制作实时摄像头检测应用。

### 1. 查询合适的模型

首先使用自然语言向 Agent 描述设备和任务需求，请 Agent 推荐合适的模型。下面的第一张图片是查询过程，第二张图片是 Agent 返回的模型选择与工作总结。

<center>
<img alt="查询合适的模型" src="../../../qcom_img/qai-appbuilder/query_suitable_model.png" width="800">
</center>

<center>
<img alt="模型查询结果与 Agent 工作总结" src="../../../qcom_img/qai-appbuilder/query_result.png" width="800">
</center>

### 2. 制作图片检测应用

在确定模型后，继续让 Agent 根据模型和图片检测需求生成应用。先查看 Agent 的请求处理过程和工作总结，再查看应用最终的效果预览。

<center>
<img alt="QAI AppBuilder 图片检测请求" src="../../../qcom_img/qai-appbuilder/pic_det_app_request.png" width="800">
</center>

<center>
<img alt="QAI AppBuilder 图片检测 Agent 工作总结" src="../../../qcom_img/qai-appbuilder/pic_det_app_result.png" width="800">
</center>

<center>
<img alt="QAI AppBuilder 图片检测应用最终效果预览" src="../../../qcom_img/qai-appbuilder/pic_det_app_preview.png" width="800">
</center>

### 3. 制作实时摄像头检测应用

图片检测应用验证完成后，可以继续让 Agent 基于前面选择的模型和应用开发结果，生成实时摄像头检测应用。同样，先查看 Agent 的请求和工作总结，最后查看应用的效果预览。

<center>
<img alt="实时摄像头检测应用请求" src="../../../qcom_img/qai-appbuilder/live_cam_det_app_request.png" width="800">
</center>

<center>
<img alt="实时摄像头检测 Agent 工作总结" src="../../../qcom_img/qai-appbuilder/live_cam_det_app_result.png" width="800">
</center>

<center>
<img alt="实时摄像头检测应用最终效果预览" src="../../../qcom_img/qai-appbuilder/live_cam_det_app_preview.png" width="800">
</center>