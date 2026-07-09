# NVIDIA 平台
## JetPack
JetPack 包括带有引导加载程序的 Jetson Linux、Linux 内核、Ubuntu 桌面环境，以及一整套用来为 GPU 计算、多媒体、图形和计算机视觉加速的库。它还包含用于主机和开发者套件的示例、文档和开发者工具，并支持更高级别的 SDK，例如用于流媒体视频分析的 DeepStream、用于机器人开发的 Isaac 以及用于对话式 AI 的 Riva。

[了解详情 JetPack 6.2](https://developer.nvidia.com/embedded/jetpack-sdk-62)

### 安装
```shell
sudo apt update
sudo apt install nvidia-jetpack
```

## TensorRT
NVIDIA TensorRT 是用于高性能深度学习推理的 API 生态系统。TensorRT 包括推理运行时和模型优化，可为生产应用提供低延迟和高吞吐量。TensorRT 生态系统包括 TensorRT、TensorRT-LLM、TensorRT 模型优化器和 TensorRT Cloud。
TensorRT 优势:
* 推理速度提升 36 倍
* 优化推理性能
* 加速各种工作负载
* 使用 Triton 进行部署、运行和扩展

[了解详情](https://developer.nvidia.cn/tensorrt)

## 边缘生成式 AI
NVIDIA Jetson Orin 可提供非凡的 AI 计算能力、大容量统一内存和全面的软件堆栈，能够以超高能效驱动最新的生成式 AI 应用。 它能够使任何由 Transformer 架构提供支持的生成式 AI 模型进行快速推理，在 MLPerf 上实现卓越的边缘性能。 通过 Jetson AI 实验室，您可以学习相关教程，了解文本生成、文本 + 视觉模型、图像生成和蒸馏技术，还可以获取在 NVIDIA Jetson Orin 上运行这些模型所需的资源。

[立即试用](https://www.jetson-ai-lab.com/)

### DeepSeek R1
#### 安装

```
sudo apt update
sudo apt install nvidia-jetpack
curl -fsSL https://ollama.com/install.sh | sh
```

#### 运行 
##### DeepSeek-R1-Distill-Qwen-1.5B
```
ollama run deepseek-r1:1.5b
```