# 部署例程

## 人工智能算法部署

AIBOX-1684 具备全面的人工智能算法本地私有化部署能力，无论是新颖、高算力需求的大语言模型，还是经典的 YOLOv5 目标检测模型，AIBOX-1684 均可胜任。

本章节主要介绍 SOPHON-DEMO 项目，大模型部署请参考[大模型部署](llm-tpu.md)这一章节。

## SOPHON-DEMO 项目简介

SOPHON-DEMO 包含了一系列主流人工智能算法的移植例程，以及详细的部署文档以便于用户运行。

项目仓库链接：[SOPHON-DEMO](https://github.com/sophgo/sophon-demo)。

## 例程清单

SOPHON-DEMO 提供的例子从易到难分为 `tutorial`、`sample`、`application` 三个模块：

* `tutorial` 模块存放一些基础接口的使用示例。
* `sample` 模块存放一些经典算法在SOPHONSDK上的串行示例。
* `application` 模块存放一些典型场景的典型应用。

| tutorial                                                                  | 编程语言    |
|---                                                                        |---         |
| [resize](https://github.com/sophgo/sophon-demo/blob/release/tutorial/resize/README.md)                                     | C++/Python |
| [crop](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop/README.md)                                         | C++/Python |
| [crop_and_resize_padding](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop_and_resize_padding/README.md)   | C++/Python |
| [ocv_jpubasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/ocv_jpubasic/README.md)                         | C++/Python |
| [ocv_vidbasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/vid_jpubasic/README.md)                         | C++/Python |
| [blend](https://github.com/sophgo/sophon-demo/blob/release/tutorial/blend/README.md)                                       | C++/Python |
| [stitch](https://github.com/sophgo/sophon-demo/blob/release/tutorial/stitch/README.md)                                     | C++/Python |

| sample                                                          | 算法类别          | 编程语言    | BModel         |
|---                                                            |---               |---          | ---           |
| [LPRNet](https://github.com/sophgo/sophon-demo/blob/release/sample/LPRNet/README.md)                           | 车牌识别          | C++/Python | FP32/FP16/INT8 |
| [ResNet](https://github.com/sophgo/sophon-demo/blob/release/sample/ResNet/README.md)                           | 图像分类          | C++/Python | FP32/FP16/INT8 |
| [RetinaFace](https://github.com/sophgo/sophon-demo/blob/release/sample/RetinaFace/README.md)                   | 人脸检测          | C++/Python | FP32           |
| [segformer](https://github.com/sophgo/sophon-demo/blob/release/sample/segformer/README.md)                     | 语义分割          | C++/Python | FP32/FP16      |
| [SAM](https://github.com/sophgo/sophon-demo/blob/release/sample/SAM/README.md)                                 | 语义分割          | Python     | FP32/FP16      |
| [yolact](https://github.com/sophgo/sophon-demo/blob/release/sample/yolact/README.md)                           | 实例分割          | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_seg](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_seg/README.md)                   | 实例分割          | C++/Python | FP32/FP16/INT8 |
| [PP-OCR](https://github.com/sophgo/sophon-demo/blob/release/sample/PP-OCR/README.md)                           | OCR              | C++/Python | FP32/FP16      | 
| [OpenPose](https://github.com/sophgo/sophon-demo/blob/release/sample/OpenPose/README.md)                       | 人体关键点检测    | C++/Python | FP32/FP16/INT8 |
| [C3D](https://github.com/sophgo/sophon-demo/blob/release/sample/C3D/README.md)                                 | 视频动作识别      | C++/Python | FP32/FP16/INT8 |
| [DeepSORT](https://github.com/sophgo/sophon-demo/blob/release/sample/DeepSORT/README.md)                       | 多目标跟踪        | C++/Python | FP32/FP16/INT8 |
| [ByteTrack](https://github.com/sophgo/sophon-demo/blob/release/sample/ByteTrack/README.md)                     | 多目标跟踪        | C++/Python | FP32/FP16/INT8 |
| [CenterNet](https://github.com/sophgo/sophon-demo/blob/release/sample/CenterNet/README.md)                     | 目标检测、姿态识别 | C++/Python | FP32/FP16/INT8 |
| [YOLOv5](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/README.md)                           | 目标检测          | C++/Python | FP32/FP16/INT8 |
| [YOLOv34](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv34/README.md)                         | 目标检测          | C++/Python | FP32/INT8      |
| [YOLOX](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOX/README.md)                             | 目标检测          | C++/Python | FP32/INT8      |
| [SSD](https://github.com/sophgo/sophon-demo/blob/release/sample/SSD/README.md)                                 | 目标检测          | C++/Python | FP32/INT8      |
| [YOLOv7](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv7/README.md)                           | 目标检测          | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_det](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_det/README.md)                   | 目标检测          | C++/Python | FP32/FP16/INT8 |
| [YOLOv5_opt](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5_opt/README.md)                   | 目标检测          | C++/Python | FP32/FP16/INT8 |
| [ppYOLOv3](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYOLOv3/README.md)                       | 目标检测          | C++/Python | FP32/FP16/INT8 |
| [ppYoloe](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYoloe/README.md)                         | 目标检测          | C++/Python | FP32/FP16      |
| [WeNet](https://github.com/sophgo/sophon-demo/blob/release/sample/WeNet/README.md)                             | 语音识别          | C++/Python | FP32/FP16      | 
| [BERT](https://github.com/sophgo/sophon-demo/blob/release/sample/BERT/README.md)                               | 语言模型          | C++/Python | FP32/FP16      | 
| [ChatGLM2](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM2/README.md)                       | 语言模型          | C++/Python | FP16/INT8/INT4 | 
| [Llama2](https://github.com/sophgo/sophon-demo/blob/release/sample/Llama2/README.md)                           | 语言模型          | C++/Python | FP16/INT8/INT4 |
| [ChatGLM3](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM3/README.md)                       | 语言模型          | Python     | FP16/INT8/INT4 | 
| [Qwen](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen/README.md)                               | 语言模型          | Python     | FP16/INT8/INT4 | 
| [Qwen1_5](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen1_5/README.md)                         | 语言模型          | Python     | FP16/INT8/INT4 | 
| [StableDiffusionV1.5](https://github.com/sophgo/sophon-demo/blob/release/sample/StableDiffusionV1_5/README.md) | 图像生成          | Python     | FP32/FP16      |
| [GroundingDINO](https://github.com/sophgo/sophon-demo/blob/release/sample/GroundingDINO/README.md)             | 多模态目标检测     | Python     | FP16           |

| application                                                    | 应用场景                  | 编程语言    | 
|---                                                             |---                       |---          | 
| [VLPR](https://github.com/sophgo/sophon-demo/blob/release/application/VLPR/README.md)                           | 多路车牌检测+识别          | C++/Python  | 
| [YOLOv5_multi](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi/README.md)           | 多路目标检测               | C++         | 
| [YOLOv5_multi_QT](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi_QT/README.md)     | 多路目标检测+QT_HDMI显示   | C++         | 

---

## 大模型部署

## LLM-TPU 项目简介

LLM-TPU 包含了各类开源生成式 AI 模型的移植部署例程，其中以 LLM 为主，也包含 Stable Diffusion（AI 绘画）。

项目仓库链接：[LLM-TPU](https://github.com/sophgo/LLM-TPU)。

## 例程清单

|Model                |INT4                |INT8                |FP16/BF16           |Huggingface Link                                                          |
|:-                   |:-                  |:-                  |:-                  |:-                                                                        |
|Baichuan2-7B         |                    |&#x2714;            |                    |[LINK](https://huggingface.co/baichuan-inc/Baichuan2-7B-Chat)             |
|ChatGLM3-6B          |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/THUDM/chatglm3-6b)                          |
|CodeFuse-7B          |&#x2714;            |&#x2714;            |                    |[LINK](https://huggingface.co/codefuse-ai/CodeFuse-DevOps-Model-7B-Chat)  |
|DeepSeek-6.7B        |&#x2714;            |&#x2714;            |                    |[LINK](https://huggingface.co/deepseek-ai/deepseek-coder-6.7b-instruct)   |
|Falcon-40B           |                    |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/tiiuae/falcon-40b)                          |
|Phi-3-mini-4k        |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct/)          |
|Qwen-7B              |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/Qwen/Qwen-7B-Chat)                          |
|Qwen-14B             |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/Qwen/Qwen-14B-Chat)                         |
|Qwen-72B             |&#x2714;            |                    |                    |[LINK](https://huggingface.co/Qwen/Qwen-72B-Chat)                         |
|Qwen1.5-0.5B         |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/Qwen/Qwen1.5-0.5B-Chat)                     |
|Qwen1.5-1.8B         |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/Qwen/Qwen1.5-1.8B-Chat)                     |
|Llama2-7B            |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf)              |
|Llama2-13B           |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/meta-llama/Llama-2-13b-chat-hf)             |
|LWM-Text-Chat        |&#x2714;            |&#x2714;            |&#x2714;            |[LINK](https://huggingface.co/LargeWorldModel/LWM-Text-Chat-1M)           |
|Mistral-7B-Instruct  |&#x2714;            |&#x2714;            |                    |[LINK](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)         |
|Stable Diffusion     |                    |                    |&#x2714;            |[LINK](https://huggingface.co/runwayml/stable-diffusion-v1-5)             |
|Stable Diffusion XL  |                    |                    |&#x2714;            |[LINK](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0)   |
|WizardCoder-15B      |&#x2714;            |                    |                    |[LINK](https://huggingface.co/WizardLM/WizardCoder-15B-V1.0)              |
|Yi-6B-chat           |&#x2714;            |&#x2714;            |                    |[LINK](https://huggingface.co/01-ai/Yi-6B-Chat)                           |
|Yi-34B-chat          |&#x2714;            |&#x2714;            |                    |[LINK](https://huggingface.co/01-ai/Yi-34B-Chat)                          |

---

## RAG 私有知识库

检索增强生成（RAG）是一种使用来自私有或专有数据源的信息来辅助文本生成的技术，该技术通过结合大型语言模型（LLM）的生成能力与从外部知识库中检索相关信息的能力，来生成更加精准和上下文相关的回答或文本内容。

通过借助 RAG 技术，可以解决大模型存在的知识时效性不足、上下文理解限制、信息来源不确定等关键性问题。

本章节提供三个可部署于 AIBOX-1684 上的 RAG 部署案例，分别为 FireflyChat、ChatDoc-TPU 和 LangChain-Chatchat-TPU。

## FireflyChat 项目简介

FireflyChat 是由 Firefly 开源团队开发的图形化大模型应用平台，部署仅需简单安装、无需编译，快速体验 RAG 对大模型的提升。

项目详情参考 [FireflyChat](fireflychat.md) 章节。

## ChatDoc-TPU 项目简介

ChatDoc-TPU 是一个完全本地化推理的文档对话工具，其主要目标是通过使用自然语言来简化与文档的交互，并提取有价值的信息。

项目仓库链接：[ChatDoc-TPU](https://github.com/wangyifan2018/ChatDoc-TPU)

## LangChain-Chatchat-TPU 项目简介

Langchain-Chatchat-TPU 是基于 Langchain-Chatchat 开发的完全本地化推理的知识库增强方案。

项目仓库链接：[LangChain-Chatchat-TPU](https://github.com/wangyifan2018/LangChain-Chatchat-TPU)

---

## 语音识别与文本转语音

## VITS-TPU 项目简介

VITS 是一个端到端的文本转语音（TTS）模型，VITS-TPU 项目实现了对 VITS 的 TPU 算法移植。

项目仓库链接：[VITS-TPU](https://github.com/wangyifan2018/VITS-TPU)

## Whisper-UI-TPU 项目简介

Whisper 是一个由 OpenAI 开发的开源语音识别（ASR）模型，它能够实现实时、多语言的语音识别。

Whisper-UI-TPU 实现了对 Whisper 的 TPU 算法移植，同时提供了便于使用的 WebUI。

项目仓库链接：[Whisper-UI-TPU](https://github.com/wangyifan2018/Whisper-UI-TPU)

---

## 图片内容检索

## CLIP-TPU 项目简介

CLIP 全称 Constrastive Language-Image Pre-training，是 OpenAI 推出的采用对比学习的文本-图像预训练模型。CLIP 在 zero-shot 文本-图像检索，zero-shot 图像分类，文生图任务 guidance，open-domain 检测分割等任务上均有非常惊艳的表现。

CLIP-TPU 项目实现了对 CLIP 的 TPU 算法移植，用户可使用该项目验证 CLIP 在 AIBOX-1684 上的实际表现。

项目仓库链接：[CLIP-TPU](https://github.com/wangyifan2018/CLIP-TPU)

## ImageSearch-tpu 项目简介

ImageSearch-tpu 项目基于 CLIP 实现了对大量图片进行内容搜索的功能，同时提供了可视化的 WebUI 便于用户使用。

项目仓库链接：[ImageSearch-tpu](https://github.com/wangyifan2018/ImageSearch-tpu)