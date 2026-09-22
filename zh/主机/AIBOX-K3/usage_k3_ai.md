# K3 AI 使用

K3 Bianbu 系统 AI 使用请点击链接跳转：[AI教程](https://spacemit.com/community/document/info?nodepath=ai/intro/root_overview.md&lang=zh)

## AI SDK 简介

SpacemiT AI SDK 是面向 K 系列芯片的 AI 应用开发套件，支持 K1/K3 的 Buildroot 与 Bianbu LXQT/GNOME 系统，主要组件包括：

* **Vision**：视觉组件，支持检测、分类、分割、跟踪、人脸、姿态等任务，支持 resnet、yolov8/yolov11 等模型
* **语音**：提供 VAD（语音活动检测）、ASR（语音识别）、TTS（语音合成）与声纹能力
* **LLM**：大语言模型推理，提供 OpenAI 兼容接口的 llama-server
* **VLM**：图文理解多模态模型，支持 FastVLM、Qwen 系列
* **RL**：机器人策略推理
* **Gateway**：统一 HTTP/WS 服务接入层

## 算力简介

**AIBOX-K3** 搭载进迭时空 SpacemiT Key Stone K3 主控芯片，采用 RISC-V 同构融合计算架构，集成 8 核高性能通用大核 X100 与 8 核超宽并行 AI 核 A100，可提供 130K DMIPS 通用算力与 60 TOPS 通用 AI 算力，支持运行 30B 参数级大模型，兼容全品类 AI 算法与模型的快速部署。

## 安装与编译

```sh
git clone --recurse-submodules https://github.com/spacemit-com/ai-sdk.git
source build/envsetup.sh
m
```

* 首次编译耗时较长，请耐心等待
* 编译产物安装到 `output/staging`
* 单组件编译：进入对应组件目录执行 `mm`
* Gateway 也可直接安装：`sudo apt install spacemit-ai-gateway`（backend 默认使用 18790 端口，console 默认使用 8326 端口）

## 常用示例

* **Vision**：先运行 `vision/scripts/download_all_models.sh` 与 `download_assets.sh` 下载模型与资源，再运行示例，例如 yolov8：

    ```sh
    yolov8 vision/examples/yolov8/config/yolov8.yaml
    ```

* **LLM**：下载 GGUF 模型后，通过 `llama-server` 启动本地服务（默认 8080 端口），使用 `llm_chat` 进行对话；使用云端服务时更换 api_base，并 `export OPENAI_API_KEY`
* **ASR/TTS/VAD**：分别运行 `asr_file_demo`、`tts_file_demo`、`vad_simple_demo`
* **Gateway 验证**：`curl localhost:18790/healthz`

## 常见问题

* 同一端口（如 8063）只能运行一个模型服务，切换模型前需先停止当前服务
* bytetrack/ocsort 实时跟踪示例需要外接屏幕
* VLM 测试中断后，使用 `pkill -f llama-server` 与 `spacemit-tcm-smi -c` 清理残留进程与 NPU 状态
* 详细参数与示例见各组件 `vision/examples/*/README.md`

## 更多资料

* [AI教程](https://spacemit.com/community/document/info?nodepath=ai/intro/root_overview.md&lang=zh)：官方 AI 文档入口，包含 AI 框架、部署方式与示例的完整说明
* [芯片产品文档](https://spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs)：包含《产品介绍》、《数据手册》和《用户手册》