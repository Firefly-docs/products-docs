# 人工智能算法部署

AIBOX-1684X 具备全面的人工智能算法本地私有化部署能力，无论是新颖、高算力需求的大语言模型，还是经典的 YOLOv5 目标检测模型，AIBOX-1684X 均可胜任。

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