# AI algorithm deployment

AIBOX-1684 has a comprehensive local private deployment capability of AI algorithms, whether it is a novel large language model with high computing requirements, or a classic YOLOv5 object detection model, AIBOX-1684 can be competent.

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