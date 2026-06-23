RK182X开发套件支持一系列主流人工智能算法的部署，支持但不限于以下算法类別：

| sample                                                                                                | 算法类别	       |
| ---------------------------------------------------------------------------------------               | -------------- |
| [LPRNet](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/LPRNet)                      | 车牌识别	       |
| [PPOCR](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/PPOCR)                        | OCR	       |
| [RetinaFace](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/RetinaFace)              | 人脸检测        |
| [clip](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/clip)                          | 多模态目标检测   |
| [deeplabv3](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/deeplabv3)                | 语义分割	       |
| [lite_transformer](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/lite_transformer)  | 轻量化NLP       |
| [mobilenet](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/mobilenet)                | 图像分类	       |
| [ppseg](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/ppseg)                        | 图像分割        |
| [ppyoloe](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/ppyoloe)                    | 目标检测        |
| [resnet](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/resnet)                      | 图像分类        |
| [whisper](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/whisper)                    | 语音识别        |
| [YAMNet](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yamnet)                      | 音频分类        |
| [yolo_world](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolo_world)              | 目标检测        |
| [yolov10](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov10)                    | 目标检测        |
| [yolov5](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov5)                      | 目标检测        |
| [yolov5_seg](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov5_seg)              | 实例分割        |
| [yolov6](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov6)                      | 目标检测        |
| [yolov7](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov7)                      | 目标检测        |
| [yolov8](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov8)                      | 目标检测        |
| [yolov8_obb](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov8_obb)              | 旋转框目标检测   |
| [yolov8_pose](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov8_pose)            | 人体关键点检测   |
| [yolov8_seg](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolov8_seg)              | 实例分割        |
| [yolox](https://github.com/airockchip/rknn_model_zoo/tree/main/examples/yolox)                        | 目标检测        |

除了以上人工智能算法，更多RK官方例程请参考[rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo)。RK182X开发套件还支持大语言模型，大语言模型部署请参考[大模型部署](usage_llm_rockchip.md)这一章节

### 1.1 NPU使用
RK182X开发套件 内置 NPU 模块, 处理性能最高可达6TOPS。使用该NPU需要下载[RKNN SDK](https://github.com/airockchip/rknn-toolkit2)，RKNN SDK 提供 C++/python 编程接口，能够帮助用户部署使用 RKNN-Toolkit2 导出的 RKNN 模型，加速 AI 应用的落地。RKNN 的整体开发步骤，流程主要分为3 个部分：模型转换、模型评估和板端部署运行。
![](../../../gs1-n2_img/npu_development_process.png)
* <font color=blue>**模型转换**</font>: 支持 `Caffe`、`TensorFlow`、`TensorFlow Lite`、`ONNX`、`DarkNet`、`PyTorch` 等模型转为 RKNN 模型，并支持 RKNN 模型导入导出，RKNN 模型能够在 Rockchip NPU 平台上加载使用。<br></br>
* <font color=blue>**模型评估**</font>: 模型评估阶段帮助用户量化和分析模型性能，包括精度、连板推理性能和内存占用等关键指标。<br></br>
*  <font color=blue>**板端部署运行**</font>: 加载 RKNN 模型到 RKNPU 平台，进行模型前处理、推理、后处理、释放等流程。<br></br>
### 1.2 RKNN-Toolkit2工具
RKNN-Toolkit2 是为用户提供在`PC`、Rockchip NPU 平台上进行模型转换、推理和性能评估的开发套件，用户通过该工具提供的 Python 接口可以便捷地完成各种操作。**注意**：RKNN-Toolkit2 开发套件运行在`PC x86_64`平台上，请勿安装在RK182X开发套件板端。

#### 1.2.1 RKNN-Toolkit2安装
RKNN SDK 提供两种RKNN-Toolkit2安装方式，通过Docker方式安装和通过pip方式安装，用户可自行选择任意一种方式进行安装。这里以pip方式为例，在`PC Ubuntu20.04(x64)`安装。因为系统中可能同时有多个版本的 Python 环境，建议使用 miniforge3 管理 Python 环境。
```
# 检查是否安装 miniforge3 和 conda 版本信息，若已安装则可省略此小节步骤。
conda -V
# 下载 miniforge3 安装包
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# 安装 miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# 进入Conda base 环境，miniforge3 为安装目录
source ~/miniforge3/bin/activate
# 创建一个 Python3.8 版本（建议版本）名为RKNN-Toolkit2 的 Conda 环境
conda create -n RKNN-Toolkit2 python=3.8
# 进入 RKNN-Toolkit2 Conda 环境
conda activate RKNN-Toolkit2
# 安装依赖库
pip3 install -r rknn-toolkit2/packages/requirements_cp38-2.0.0b0.txt
# 安装 RKNN-Toolkit2，如
pip3 install rknn-toolkit2/packages/rknn_toolkit2-2.0.0b0+9bab5682-cp38-cp38-linux_x86_64.whl
```
若执行以下命令没有报错，则安装成功。
```
python
from rknn.api import RKNN
```
若安装失败或需要其他安装方式请查阅[RKNN SDK文档](https://github.com/airockchip/rknn-toolkit2/tree/master/doc)。

#### 1.2.2 模型转换Demo
在rknn-toolkit2/examples下有各种功能的 Demo ，这里我们运行一个模型转换 Demo 为例子，这个 Demo 展示了在 PC 上将 yolov5 onnx 模型转换成 RKNN 模型，然后导出、在仿真器上推理的过程。模型转换的具体实现请参考 Demo 内源代码以及RKNN SDK文档。
```
root@9893c1c48f42:/rknn-toolkit2/examples/onnx/yolov5# python3 test.py 
I rknn-toolkit2 version: 2.0.0b0+9bab5682
--> Config model
done
--> Loading model
I It is recommended onnx opset 19, but your onnx model opset is 12!
I Model converted from pytorch, 'opset_version' should be set 19 in torch.onnx.export for successful convert!
I Loading : 100%|██████████████████████████████████████████████| 125/125 [00:00<00:00, 22152.70it/s]
done
--> Building model
I GraphPreparing : 100%|███████████████████████████████████████| 149/149 [00:00<00:00, 10094.68it/s]
I Quantizating : 100%|███████████████████████████████████████████| 149/149 [00:00<00:00, 428.06it/s]
W build: The default input dtype of 'images' is changed from 'float32' to 'int8' in rknn model for performance!
                       Please take care of this change when deploy rknn model with Runtime API!                                                                                                                      
W build: The default output dtype of 'output' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
W build: The default output dtype of '283' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
W build: The default output dtype of '285' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
I rknn building ...
I rknn buiding done.
done
--> Export rknn model
done
--> Init runtime environment
I Target is None, use simulator!
done
--> Running model
I GraphPreparing : 100%|███████████████████████████████████████| 153/153 [00:00<00:00, 10289.55it/s]
I SessionPreparing : 100%|██████████████████████████████████████| 153/153 [00:00<00:00, 1926.55it/s]
done
   class        score      xmin, ymin, xmax, ymax
--------------------------------------------------
   person       0.884     [ 208,  244,  286,  506]
   person       0.868     [ 478,  236,  559,  528]
   person       0.825     [ 110,  238,  230,  533]
   person       0.334     [  79,  353,  122,  517]
    bus         0.705     [  92,  128,  554,  467]
Save results to result.jpg!
```
如果 rknn-toolkit2 缺少您需要的模型Demo，可以参考[rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo)仓库的python例程。
### 1.3 RKNPU2使用
RKNPU2 提供在`板端Rockchip NPU 平台`上进行模型推理的C/C++编程接口。
#### 1.3.1 环境安装
RKNPU2 Runtime 库文件`librknnrt.so`，若板端缺少该文件或需要更新，对于Linux系统可以将`rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so`通过`scp`推送至`/usr/lib`目录。
#### 1.3.2 板端推理
在RKNN SDK 目录`rknpu2/examples`提供了许多模型推理Demo，用户可以参考例程开发部署自己的AI应用。[rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo)仓库同样提供了C/C++的例程

### 1.4 RKNN-Toolkit Lite2介绍
RKNN-Toolkit Lite2 为用户提供`板端模型推理`的 `Python` 接口，方便用户使用 Python 语言进行 AI 应用开发。**注意**：RKNN-Toolkit Lite2安装运行在`板端`，且仅作推理使用，无法转换模型。
#### 1.4.1 RKNN-Toolkit Lite2安装
```
# 安装python3/pip3
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
# RKNN-Toolkit Lite2安装，在rknn-toolkit-lite2/packages找到安装包，根据系统python版本安装对应版本
pip3 install rknn_toolkit_lite2-2.0.0b0-cp310-cp310-linux_aarch64.whl
```
#### 1.4.2 RKNN-Toolkit Lite2使用
在 RKNN SDK/rknn-toolkit-lite2/examples 目录，有基于RKNN-Toolkit Lite2 开发的应用，虽然提供的例程数量有限，但是实际上`RKNN-Toolkit Lite2` 和 `RKNN-Toolkit2`的接口使用是十分类似的，用户完全可以参考RKNN-Toolkit2例程移植到RKNN-Toolkit Lite2。

### 1.5 详细开发文档
NPU 及 Toolkit 详细使用方法请参考RKNN SDK下`doc`文档。

### 1.6 FAQs
**Q1：rknn模型推理精度对比原模型精度下降？**

A1：请参考《Rockchip_RKNPU_User_Guide_RKNN_SDK*》文档的精度排查章节逐步查找原因。

**Q2：转换时提示 Expand 算子不支持？**

A2：尝试更新 RKNN-Toolkit2 / RKNPU2 至最新版本或者采用 repeat 算子来替代 expand 算子。

**更多转换问题或者报错原因可以参考《Rockchip_RKNPU_User_Guide_RKNN_SDK\*》文档常见问题章节**。
