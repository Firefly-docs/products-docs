# NVIDIA 
## JetPack
JetPack includes Jetson Linux with bootloader, Linux kernel, Ubuntu desktop environment, and a complete set of libraries for GPU computing, multimedia, graphics, and computer vision acceleration. It also includes examples, documentation, and developer tools for hosts and developer suites, and supports higher-level SDKs such as DeepStream for streaming video analysis, Isaac for robot development, and Riva for conversational AI.

[Learn More JetPack 6.2](https://developer.nvidia.com/embedded/jetpack-sdk-62)

### Install
```shell
sudo apt update
sudo apt install nvidia-jetpack
```

## TensorRT
NVIDIA TensorRT is an ecosystem of APIs for high-performance deep learning inference. TensorRT includes an inference runtime and model optimizations that deliver low latency and high throughput for production applications. The TensorRT ecosystem includes TensorRT, TensorRT-LLM, TensorRT Model Optimizer, and TensorRT Cloud.
TensorRT Advantage:
* Speed Up Inference by 36X
* Optimize Inference Performance
* Accelerate Every Workload
* Deploy, Run, and Scale With Triton

[Learn More TensorRT](https://developer.nvidia.cn/tensorrt)

## Generative AI
NVIDIA Jetson Orin gives you unparalleled AI compute, large unified memory, and comprehensive software stacks, delivering superior energy efficiency to drive the latest generative AI applications. It delivers fast inference for any gen AI models powered by the transformer architecture, leading the edge performance on MLPerf. The Jetson AI Lab is your gateway to explore tutorials on text generation, text + vision models, image generation, distillation techniques, and access resources to run these models on NVIDIA Jetson Orin. Join the generative AI revolution and start today.

[Try it Now](https://www.jetson-ai-lab.com/)

### DeepSeek R1
#### Install
```
sudo apt update
sudo apt install nvidia-jetpack
curl -fsSL https://ollama.com/install.sh | sh
```

#### Run
##### DeepSeek-R1-Distill-Qwen-1.5B
```
ollama run deepseek-r1:1.5b
```