# 编译 RK1820/RK1828 安装包

## 获取 SDK

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
比如，SDK 压缩包是 `RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.0.4.tgz`。

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.0.4.tgz
.repo/repo/repo sync -l
```

## 配置 
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

## 编译
```
./build.sh
```

生成的软件安装包在 `output/firmware/rknn3_rk182x_m2_installer_arm64.tgz`

## 安装
手动安装 RK1820/RK1828 软件包，按如下步骤操作：
* 拷贝 `rknn3_rk182x_m2_installer_arm64.tgz` 到主控端
* 解压 `tar xzf rknn3_rk182x_m2_installer_arm64.tgz`
* 安装 `./install.sh`
    * 安装重启后， RK3588 或者 RK3576 端系统会在启动后， ⾃动下载 RK182X 的固件，并启动后台服务程序。


## 其他
### 版本 V1.0.4
```
sudo rknn-smi -v
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.0
RC chips connect version      : 3.3.1
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.0.4
rknn3 API version             : NA
```

## FAQ

### 当前支持的模型
| 模型名称 | 模型来源 |
|---------|---------|
| Qwen2.5-0.5B | https://huggingface.co/Qwen/Qwen2.5-0.5B |
| Qwen2.5-3B | https://huggingface.co/Qwen/Qwen2.5-3B-Instruct |
| Qwen2.5-7B | https://huggingface.co/Qwen/Qwen2.5-7B-Instruct |
| Qwen3-0.6B | https://huggingface.co/Qwen/Qwen3-0.6B |
| Qwen3-1.7B | https://huggingface.co/Qwen/Qwen3-1.7B |
| Qwen3-4B | https://huggingface.co/Qwen/Qwen3-4B |
| Qwen3-8B | https://huggingface.co/Qwen/Qwen3-8B |
| HY-MT1.5-1.8B | https://huggingface.co/tencent/HY-MT1.5-1.8B |
| Youtu-LLM-2B | https://huggingface.co/tencent/Youtu-LLM-2B |
| GLM-Edge-1.5B-Chat | https://modelscope.cn/models/ZhipuAI/glm-edge-1.5b-chat |
| Qwen2.5-VL-3B | https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct |
| Qwen2.5-VL-7B | https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct |
| Qwen2.5-Omni-3B (Thinker) | https://huggingface.co/Qwen/Qwen2.5-Omni-3B |
| Qwen3-VL-2B | https://huggingface.co/Qwen/Qwen3-VL-2B-Instruct |
| Qwen3-VL-4B | https://huggingface.co/Qwen/Qwen3-VL-4B-Instruct |
| FastVLM | https://github.com/apple/ml-fastvlm |
| InternVL3-2B | https://huggingface.co/OpenGVLab/InternVL3-2B |
| InternVL3_5-4B | https://huggingface.co/OpenGVLab/InternVL3_5-4B-Instruct |
| MiMo-VL-7B-RL | https://huggingface.co/XiaomiMiMo/MiMo-VL-7B-RL |
| Gemma-4-E2B | https://huggingface.co/google/gemma-4-E2B-it |
| Gemma-4-E4B | https://huggingface.co/google/gemma-4-E4B-it |
| SmolVLM-500M-Instruct | https://huggingface.co/HuggingFaceTB/SmolVLM-500M-Instruct |
| SmolVLM2-500M-Video-Instruct | https://huggingface.co/HuggingFaceTB/SmolVLM2-500M-Video-Instruct |
| UI-TARS-2B-SFT | https://huggingface.co/ByteDance-Seed/UI-TARS-2B-SFT |
| PaddleOCR VL | https://huggingface.co/PaddlePaddle/PaddleOCR-VL |
| Qwen3-Reranker-0.6B | https://huggingface.co/Qwen/Qwen3-Reranker-0.6B |
| Qwen3-Reranker-4B | https://huggingface.co/Qwen/Qwen3-Reranker-4B |
| Qwen3-Embedding-0.6B | https://huggingface.co/Qwen/Qwen3-Embedding-0.6B |
| Qwen3-Embedding-4B | https://huggingface.co/Qwen/Qwen3-Embedding-4B |
| gme-Qwen2-VL-2B-Instruct | https://huggingface.co/Alibaba-NLP/gme-Qwen2-VL-2B-Instruct |
| Qwen3-ASR-0.6B | https://huggingface.co/Qwen/Qwen3-ASR-0.6B |
| Qwen3-TTS-12Hz-1.7B | https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base |
| VITS | https://github.com/jaywalnut310/vits |
| Whisper | https://huggingface.co/openai/whisper-large-v3 |
| SenseVoiceSmall | https://modelscope.cn/models/iic/SenseVoiceSmall |
| Zipformer | https://huggingface.co/pfluo/k2fsa-zipformer-chinese-english-mixed |
| SigLIP | https://huggingface.co/google/siglip-so400m-patch14-384 |
| Siglip2-so400m | https://huggingface.co/google/siglip2-so400m-patch14-384 |
| MetaCLIP2 | https://huggingface.co/facebook/metaclip-2-worldwide-m16-384 |
| Dinov3 | https://huggingface.co/facebook/dinov3-vits16-pretrain-lvd1689m |
| Depth-Anything-V2-small | https://huggingface.co/depth-anything/Depth-Anything-V2-Small |
| GR00T-N1.6-3B | https://huggingface.co/nvidia/GR00T-N1.6-3B |
| MobilenetV1 | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/mobilenet_v1/mobilenet_v1_1.0_224.tflite |
| MobilenetV2 | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/mobilenet/mobilenetv2-12.onnx |
| Resnet50V2 | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/resnet/resnet50-v2-7.onnx |
| YOLOv5s | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/yolov5/yolov5s_rknn3.onnx |
| YOLOv6s | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/yolov6/yolov6s_rknn3.onnx |
| YOLOv8s | https://ftrg.zbox.filez.com/v2/delivery/data/95f00b0fc900458ba134f8b180b3f7a1/examples/yolov8/yolov8s_rknn3.onnx |

具体可参考SDK中 SDK_Path/rknn/rknn3-runtime/doc/00_Rockchip_RKNPU3_ReleaseNote_RKNN3_SDK_V1.0.4_CN.pdf , 里面有详细说明。