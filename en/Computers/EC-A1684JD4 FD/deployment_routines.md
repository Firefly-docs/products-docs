# Deployment Routines

## AI algorithm deployment

EC-A1684JD4 FD has a comprehensive local private deployment capability of AI algorithms, whether it is a novel large language model with high computing requirements, or a classic YOLOv5 object detection model, EC-A1684JD4 FD can be competent.

This chapter mainly introduces the SOPHON-DEMO project. For large model deployment, refer to the [LLM deployment](llm-tpu.md) chapter.

## SOPHON-DEMO project introduction

SOPHON-DEMO contains a series of porting routines for mainstream AI algorithms, as well as detailed deployment documentation to make it easy for users to run.

Project repository link: [SOPHON-DEMO](https://github.com/sophgo/sophon-demo)。

## Examples list

The examples provided by SOPHON-DEMO are divided into three modules from easy to difficult: `tutorial`, `sample` and `application`.

* `tutorial` module stores some examples of basic interfaces.
* `sample` module stores some serial examples of classic algorithms on SOPHONSDK.
* `application` module stores some typical applications in typical scenarios.

| tutorial                                                                  | code    |
|---                                                                        |---         |
| [resize](https://github.com/sophgo/sophon-demo/blob/release/tutorial/resize/README.md)                                     | C++/Python |
| [crop](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop/README.md)                                         | C++/Python |
| [crop_and_resize_padding](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop_and_resize_padding/README.md)   | C++/Python |
| [ocv_jpubasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/ocv_jpubasic/README.md)                         | C++/Python |
| [ocv_vidbasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/vid_jpubasic/README.md)                         | C++/Python |
| [blend](https://github.com/sophgo/sophon-demo/blob/release/tutorial/blend/README.md)                                       | C++/Python |
| [stitch](https://github.com/sophgo/sophon-demo/blob/release/tutorial/stitch/README.md)                                     | C++/Python |

| contents                                                      | category                           | code       |  BModel       |
|---                                                            |---                                 |---          | ---           |
| [LPRNet](https://github.com/sophgo/sophon-demo/blob/release/sample/LPRNet/README.md)                           | License Plate Recognition          | C++/Python | FP32/FP16/INT8 |
| [ResNet](https://github.com/sophgo/sophon-demo/blob/release/sample/ResNet/README.md)                           | Image classification               | C++/Python | FP32/FP16/INT8 |
| [RetinaFace](https://github.com/sophgo/sophon-demo/blob/release/sample/RetinaFace/README.md)                   | Face detection                     | C++/Python | FP32           |
| [segformer](https://github.com/sophgo/sophon-demo/blob/release/sample/segformer/README.md)                     | Semantic segmentation              | C++/Python | FP32/FP16      |
| [SAM](https://github.com/sophgo/sophon-demo/blob/release/sample/SAM/README.md)                                 | Semantic segmentation              | Python     | FP32/FP16      |
| [yolact](https://github.com/sophgo/sophon-demo/blob/release/sample/yolact/README.md)                           | Instance segmentation              | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_seg](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_seg/README.md)                   | Instance segmentation              | C++/Python | FP32/FP16/INT8 |
| [PP-OCR](https://github.com/sophgo/sophon-demo/blob/release/sample/PP-OCR/README.md)                           | OCR                                | C++/Python | FP32/FP16      |
| [OpenPose](https://github.com/sophgo/sophon-demo/blob/release/sample/OpenPose/README.md)                       | Keypoint detection                 | C++/Python | FP32/FP16/INT8 |
| [C3D](https://github.com/sophgo/sophon-demo/blob/release/sample/C3D/README.md)                                 | Video recognition                  | C++/Python | FP32/FP16/INT8 |
| [DeepSORT](https://github.com/sophgo/sophon-demo/blob/release/sample/DeepSORT/README.md)                       | Object tracking                    | C++/Python | FP32/FP16/INT8 |
| [ByteTrack](https://github.com/sophgo/sophon-demo/blob/release/sample/ByteTrack/README.md)                     | Object tracking                    | C++/Python | FP32/FP16/INT8 |
| [CenterNet](https://github.com/sophgo/sophon-demo/blob/release/sample/CenterNet/README.md)                     | Object Detection + pose estimation | C++/Python | FP32/FP16/INT8 |
| [YOLOv5](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/README.md)                           | Object Detection                   | C++/Python | FP32/FP16/INT8 |
| [YOLOv34](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv34/README.md)                         | Object Detection                   | C++/Python | FP32/INT8      |
| [YOLOX](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOX/README.md)                             | Object Detection                   | C++/Python | FP32/INT8      |
| [SSD](https://github.com/sophgo/sophon-demo/blob/release/sample/SSD/README.md)                                 | Object Detection                   | C++/Python | FP32/INT8      |
| [YOLOv7](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv7/README.md)                           | Object Detection                   | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_det](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_det/README.md)                   | Object Detection                   | C++/Python | FP32/FP16/INT8 |
| [YOLOv5_opt](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5_opt/README.md)                   | Object Detection                   | C++/Python | FP32/FP16/INT8 |
| [ppYOLOv3](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYOLOv3/README.md)                       | Object Detection                   | C++/Python | FP32/FP16/INT8 |
| [ppYoloe](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYoloe/README.md)                         | Object Detection                   | C++/Python | FP32/FP16      |
| [WeNet](https://github.com/sophgo/sophon-demo/blob/release/sample/WeNet/README.md)                             | Speech Recognition                 | C++/Python | FP32/FP16      |
| [BERT](https://github.com/sophgo/sophon-demo/blob/release/sample/BERT/README.md)                               | Language                           | C++/Python | FP32/FP16      |
| [ChatGLM2](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM2/README.md)                       | Language                           | C++/Python | FP16/INT8/INT4 |
| [Llama2](https://github.com/sophgo/sophon-demo/blob/release/sample/Llama2/README.md)                           | Language                           | C++        | FP16/INT8/INT4 |
| [ChatGLM3](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM3/README.md)                       | Language                           | Python     | FP16/INT8/INT4 | 
| [Qwen](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen/README.md)                               | Language                           | Python     | FP16/INT8/INT4 | 
| [Qwen1_5](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen1_5/README.md)                         | Language                           | Python     | FP16/INT8/INT4 | 
| [StableDiffusionV1.5](https://github.com/sophgo/sophon-demo/blob/release/sample/StableDiffusionV1_5/README.md) | Image Generation                   | Python     | FP32/FP16      |
| [GroundingDINO](https://github.com/sophgo/sophon-demo/blob/release/sample/GroundingDINO/README.md)             | MultiModal Object Detection        | Python     | FP16           |

| application                                                    | scenarios                 | code    | 
|---                                                             |---                       |---          | 
| [VLPR](https://github.com/sophgo/sophon-demo/blob/release/application/VLPR/README.md)                           | Multi-streams Vehicle License Plate Recognition | C++/Python  | 
| [YOLOv5_multi](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi/README.md)           | Multi-streams Object Detection       | C++         | 
| [YOLOv5_multi_QT](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi_QT/README.md)     | Multi-streams Object Detection + QT_HDMI display    | C++         | 

---

## LLM deployment

## LLM-TPU project introduction

LLM-TPU contains migration deployment routines for various open source generative AI models, mainly LLM, also Stable Diffusion (AI painting).

Project repository link: [LLM-TPU](https://github.com/sophgo/LLM-TPU).

## Examples list

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

## RAG private knowledge base

Retrieval-Augmented Generation (RAG) is a technology that utilizes information from private or proprietary data sources to enhance text generation. It combines the generative capabilities of large language models (LLMs) with the ability to retrieve relevant information from external knowledge bases, thereby producing more accurate and contextually relevant responses or text content.

By leveraging RAG techniques, key issues inherent in large models such as outdated knowledge, limitations in contextual understanding, and uncertainty of information sources can be addressed.

This section provides three RAG deployment cases that can be deployed on EC-A1684JD4 FD, namely FireflyChat, ChatDoc-TPU and LangChain-Chatchat-TPU.

## FireflyChat project introduction

FireflyChat is a graphical application platform for LLM developed by the Firefly team. It requires only simple installation with no need for compilation, allowing for quick experience of the enhancement that RAG brings to LLM.

For details, see [FireflyChat](fireflychat.md).

## ChatDoc-TPU project introduction

ChatDoc-TPU is a fully localized inference document conversation tool whose main goal is to simplify interactions with documents and extract valuable information by using natural language.

Project repository link: [ChatDoc-TPU](https://github.com/wangyifan2018/ChatDoc-TPU)

## LangChain-Chatchat-TPU project introduction

LangChain-Chatchat-TPU is a fully localized inference knowledge base enhancement scheme based on Langchain-Chatchat.

Project repository link: [LangChain-Chatchat-TPU](https://github.com/wangyifan2018/LangChain-Chatchat-TPU)

---

## TTS and ASR

## VITS-TPU project introduction

VITS is an end-to-end text-to-speech (TTS) model. The VITS-TPU project implements TPU algorithm porting to VITS.

Project repository link: [VITS-TPU](https://github.com/wangyifan2018/VITS-TPU)

## Whisper-UI-TPU project introduction

Whisper is an open source speech recognition (ASR) model developed by OpenAI that enables real-time, multilingual speech recognition.

The Whisper-UI-TPU implements TPU algorithm porting to Whisper, and also provides an easy-to-use WebUI.

Project repository link: [Whisper-UI-TPU](https://github.com/wangyifan2018/Whisper-UI-TPU)

---

## Image content retrieval

## CLIP-TPU project introduction

CLIP, which stands for Constrastive Language-Image Pre-training, is a text-image pre-training model implemented by OpenAI using contrast learning. CLIP has a very impressive performance in zero-shot text-image retrieval, zero-shot image classification, Vincennes chart task guidance, open-domain detection segmentation and other tasks.

The CLIP-TPU project implements the TPU algorithm transplantation of CLIP, and users can use this project to verify the actual performance of CLIP on EC-A1684JD4 FD.

Project repository link: [CLIP-TPU](https://github.com/wangyifan2018/CLIP-TPU)

## ImageSearch-tpu project introduction

ImageSearch-tpu project realizes the function of searching a large number of images based on CLIP, and provides a visual WebUI for users to use.

Project repository link: [ImageSearch-tpu](https://github.com/wangyifan2018/ImageSearch-tpu)