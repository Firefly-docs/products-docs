# Artificial Intelligence Algorithm Deployment Routine

AIBOX-1688X has comprehensive capabilities for local private deployment of artificial intelligence algorithms. Whether it's a novel, high-computational-demand large language model or the classic YOLOv5 object detection model, AIBOX-1688X can handle both.

This chapter mainly introduces the SOPHON-DEMO project. For large model deployment, please refer to the [Large Model Deployment Routine](llm-tpu.md) section.

## SOPHON-DEMO Project Introduction

SOPHON-DEMO includes a series of transplantation routines for mainstream artificial intelligence algorithms, as well as detailed deployment documentation to facilitate user operation. Project repository: [SOPHON-DEMO](https://github.com/sophgo/sophon-demo).

## Routine List

The examples provided by SOPHON-DEMO are categorized from easy to difficult into three modules: `tutorial`, `sample`, and `application`:
* The `tutorial` module contains examples of basic interface usage.
* The `sample` module contains serial examples of classic algorithms on SOPHONSDK.
* The `application` module contains typical applications for typical scenarios.

| tutorial                                                                  | programming language    |
|---                                                                        |---         |
| [resize](https://github.com/sophgo/sophon-demo/blob/release/tutorial/resize/README.md)                                     | C++/Python |
| [crop](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop/README.md)                                         | C++/Python |
| [crop_and_resize_padding](https://github.com/sophgo/sophon-demo/blob/release/tutorial/crop_and_resize_padding/README.md)   | C++/Python |
| [ocv_jpubasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/ocv_jpubasic/README.md)                         | C++/Python |
| [ocv_vidbasic](https://github.com/sophgo/sophon-demo/blob/release/tutorial/vid_jpubasic/README.md)                         | C++/Python |
| [blend](https://github.com/sophgo/sophon-demo/blob/release/tutorial/blend/README.md)                                       | C++/Python |
| [stitch](https://github.com/sophgo/sophon-demo/blob/release/tutorial/stitch/README.md)                                     | C++/Python |

| sample | Algorithm Category | Programming Language | BModel |
|--- |--- |--- |--- | --- |
| [LPRNet](https://github.com/sophgo/sophon-demo/blob/release/sample/LPRNet/README.md) | Licence Plate Recognition | C++/Python | FP32/FP16/INT8 |
| [ResNet](https://github.com/sophgo/sophon-demo/blob/release/sample/ResNet/README.md) | Image Classification | C++/Python | FP32/FP16/INT8 |
| [RetinaFace](https://github.com/sophgo/sophon-demo/blob/release/sample/RetinaFace/README.md) | Face Detection | C++/Python | FP32 |
| [segformer](https://github.com/sophgo/sophon-demo/blob/release/sample/segformer/README.md) | Semantic Segmentation | C++/Python | FP32/FP16 |
| [SAM](https://github.com/sophgo/sophon-demo/blob/release/sample/SAM/README.md) | Semantic Segmentation | Python | FP32/FP16 |
| [yolact](https://github.com/sophgo/sophon-demo/blob/release/sample/yolact/README.md) | Instance Segmentation | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_seg](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_seg/README.md) | Example splitting | C++/Python | FP32/FP16/INT8 |
| [PP-OCR](https://github.com/sophgo/sophon-demo/blob/release/sample/PP-OCR/README.md) | OCR | C++/Python | FP32/FP16 | 
| [OpenPose](https://github.com/sophgo/sophon-demo/blob/release/sample/OpenPose/README.md) | Human Body Keypoint Detection | C++/Python | FP32/FP16/INT8 |
| [C3D](https://github.com/sophgo/sophon-demo/blob/release/sample/C3D/README.md) | Video Motion Recognition | C++/Python | FP32/FP16/INT8 |
| [DeepSORT](https://github.com/sophgo/sophon-demo/blob/release/sample/DeepSORT/README.md) | Multi-target Tracking | C++/Python | FP32/FP16/INT8 |
| [ByteTrack](https://github.com/sophgo/sophon-demo/blob/release/sample/ByteTrack/README.md) | Multi-target Tracking | C++/Python | FP32/FP16/INT8 |
| [CenterNet](https://github.com/sophgo/sophon-demo/blob/release/sample/CenterNet/README.md) | Target Detection, Attitude Recognition | C++/Python | FP32/FP16/INT8 |
| [YOLOv5](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/README.md) | Target Detection | C++/Python | FP32/FP16/INT8 |
| [YOLOv34](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv34/README.md) | Target Detection | C++/Python | FP32/INT8 |
| [YOLOX](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOX/README.md) | Target Detection | C++/Python | FP32/INT8 |
| [SSD](https://github.com/sophgo/sophon-demo/blob/release/sample/SSD/README.md) | Target Detection | C++/Python | FP32/INT8 |
| [YOLOv7](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv7/README.md) | Target Detection | C++/Python | FP32/FP16/INT8 |
| [YOLOv8_det](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv8_det/README.md) | Target Detection | C++/Python | FP32/FP16/INT8 |
| [YOLOv5_opt](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5_opt/README.md) | Target Detection | C++/Python | FP32/FP16/INT8 |
| [ppYOLOv3](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYOLOv3/README.md) | Target Detection | C++/Python | FP32/FP16/INT8 |
| [ppYoloe](https://github.com/sophgo/sophon-demo/blob/release/sample/ppYoloe/README.md) | Target Detection | C++/Python | FP32/FP16 |
| [WeNet](https://github.com/sophgo/sophon-demo/blob/release/sample/WeNet/README.md) | Speech Recognition | C++/Python | FP32/FP16 | 
| [BERT](https://github.com/sophgo/sophon-demo/blob/release/sample/BERT/README.md) | Language Modelling | C++/Python | FP32/FP16 | 
| [ChatGLM2](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM2/README.md) | Language Models | C++/Python | FP16/INT8/INT4 | 
| [Llama2](https://github.com/sophgo/sophon-demo/blob/release/sample/Llama2/README.md) | Language Models | C++/Python | FP16/INT8/INT4 |
| [ChatGLM3](https://github.com/sophgo/sophon-demo/blob/release/sample/ChatGLM3/README.md) | Language Models | Python | FP16/INT8/INT4 | 
| [Qwen](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen/README.md) | Language Models | Python | FP16/INT8/INT4 | 
| [Qwen1_5](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen1_5/README.md) | Language Models | Python | FP16/INT8/INT4 | [ StableDiffusionV1.5](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen1_5/README.md) 
| [StableDiffusionV1.5](https://github.com/sophgo/sophon-demo/blob/release/sample/StableDiffusionV1_5/README.md) | Image Generation | Python | FP32/ FP16 | | [GroundingDINO]()
| [GroundingDINO](https://github.com/sophgo/sophon-demo/blob/release/sample/GroundingDINO/README.md) | Multimodal Target Detection | Python | FP16 | Applications | Application Scenarios

| Applications | Application Scenarios | Programming Languages 
|--- |--- |--- | 
| [VLPR](https://github.com/sophgo/sophon-demo/blob/release/application/VLPR/README.md) | Multiple Licence Plate Detection + Recognition | C++/Python | [YOLOv5 
| [YOLOv5_multi](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi/README.md) | Multiple Licence Plate Detection + Recognition | C++ | [YOLOv5_multi ]() 
| [YOLOv5_multi_QT](https://github.com/sophgo/sophon-demo/blob/release/application/YOLOv5_multi_QT/README.md) | multiplexed target detection + qt_hdmi display | c++ | [YOLOv5_multi_QT]() 