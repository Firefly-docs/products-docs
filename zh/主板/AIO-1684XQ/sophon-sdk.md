# Sophon SDK 开发

## Sophon SDK

Sophon SDK是算能科技基于其自主研发的 AI 芯片所定制的深度学习 SDK，涵盖了神经网络推理阶段所需的模型优化、高效运行支持等能力，为深度学习应用开发和部署提供易用、高效的全栈式解决方案。

Sophon SDK既兼容第三代BM1684芯片，也支持第四代BM1684X芯片。


### Sophon SDK 包组成

Sophon SDK 包括基础工具包与高阶工具包。

#### 基础工具包

包括：

* tpu-nntc 负责对第三方深度学习框架下训练得到的神经网络模型进行离线编译和优化，生成最终运行时需要的BModel。目前支持Caffe、Darknet、MXNet、ONNX、PyTorch、PaddlePaddle、TensorFlow等。
* libsophon 提供BMCV、BMRuntime、BMLib等库，用来驱动VPP、TPU等硬件，完成图像处理、张量运算、模型推理等操作，供用户进行深度学习应用开发。
* sophon-mw 封装了BM-OpenCV、BM-FFmpeg等库，用来驱动VPU、JPU等硬件，支持RTSP流、GB28181流的解析，视频图像编解码加速等，供用户进行深度学习应用开发。
* sophon-sail 提供了支持Python/C++的高级接口，是对BMRuntime、BMCV、BMDecoder、BMLib等底层库接口的封装，供用户进行深度学习应用开发。

#### 高阶工具包
包括：
* tpu-mlir 为TPU编译器工程提供一套完整的工具链，可以将不同框架下预训练的神经网络，转化为可以在算能TPU上高效运行的二进制文件BModel。目前直接支持的框架包括tflite、onnx和Caffe。
* tpu-perf 为模型性能和精度验证提供了一套完整工具包。
* tpu-kernel 是芯片底层开发接口，既可以调用专用指令实现深度学习业务逻辑的加速，又可以调用通用指令实现客制的各种算法加速。

![../_images/new_SophonSDK.png](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/zh/html/_images/new_SophonSDK.png)

### Sophon SDK 目录

Sophon SDK 提供了十一个文件夹模块，具体如下表所示：

| 文件夹名        | 备注                                                      |
| ----------------- | ----------------------------------------------------------- |
| libsophon       | 支持sophon设备硬件加速的图像处理、张量运算、模型推理等库  |
| sophon-mw       | 支持sophon设备硬件加速的多媒体库                          |
| tpu-kernel      | 底层开发接口                                              |
| tpu-nntc        | TPU编译器工具链                                           |
| tpu-mlir        | TPU编译器工具链                                           |
| tpu-perf        | 模型性能和精度验证工具包                                  |
| sophon-pipeline | 基于pipeline的高性能推理框架                              |
| sophon-img      | SoC模式安装包                                             |
| sophon-demo     | 针对单模型或者场景的综合例程                              |
| sophon-sail     | 对底层接口进行C++/Python API封装的接口库                  |
| sophon-rpc      | 在PCIe卡产品上将一些计算任务offload到卡上的ARM CPU 上进行 |

注：sophon-sail 的安装包可从[下载中心]获取。

### Sophon SDK 解压后的目录
解压后的 SDK 文件结构如下：

```
  1SophonSDK
  2
  3├── libsophon_<date>_<hash>
  4│   ├── BMCV_Technical_Reference_Manual.pdf
  5│   ├── BMCV开发参考手册.pdf
  6│   ├── BMLib_Technical_Reference_Manual.pdf
  7│   ├── BMLIB开发参考手册.pdf
  8│   ├── BMRuntime Technical Reference Manual.pdf
  9│   ├── BMRUNTIME开发参考手册.pdf
 10│   ├── libsophon_<x.y.z>_aarch64.tar.gz                           #arm64机器，其他LINUX系统对应的libsophon安装包
 11│   ├── libsophon_<x.y.z>_x86_64.tar.gz                            #x86_64机器，其他LINUX系统对应的libsophon安装包
 12│   ├── libsophon_dockerfile
 13│   ├── libsophon.MD5
 14│   ├── LIBSOPHON_User_Guide.pdf
 15│   ├── LIBSOPHON使用手册.pdf
 16│   ├── release_version.txt
 17│   ├── sophon-driver_<x.y.z>_amd64.deb                            #x86_64机器，Debian/Ubuntu系统对应的PCIe卡驱动安装文件
 18│   ├── sophon-driver_<x.y.z>_arm64.deb                            #arm64机器，Debian/Ubuntu系统对应的PCIe卡驱动安装文件
 19│   ├── sophon-libsophon_<x.y.z>_amd64.deb                         #x86_64机器，Debian/Ubuntu系统对应的libsophon运行时环境安装文件
 20│   ├── sophon-libsophon_<x.y.z>_arm64.deb                         #arm64机器，Debian/Ubuntu系统对应的libsophon运行时环境安装文件
 21│   ├── sophon-libsophon-dev_<x.y.z>_amd64.deb                     #x86_64机器，Debian/Ubuntu系统对应的libsophon开发环境安装文件
 22│   └── sophon-libsophon-dev_<x.y.z>_arm64.deb                     #arm64机器，Debian/Ubuntu系统对应的libsophon开发环境安装文件
 23├── sophon-demo_<date>_<hash>
 24│   ├── release_version.txt
 25│   ├── sophon-demo.MD5
 26│   └── sophon-demo_v<x.y.z>_b909566_20221027.tar.gz               #针对单模型或场景的综合例程
 27├── sophon-img_<date>_<hash>
 28│   ├── bsp-debs
 29│   ├── bsp_update.tgz
 30│   ├── libsophon_soc_<x.y.z>_aarch64.tar.gz                       #交叉编译所需文件
 31│   ├── release_version.txt
 32│   ├── sdcard.tgz                                                 #SoC卡刷包
 33│   ├── SOPHON_BSP_Technical_Reference_Manual.pdf
 34│   ├── SOPHON BSP开发参考手册.pdf
 35│   ├── sophon-img.MD5
 36│   ├── sophon-soc-libsophon_<x.y.z>_arm64.deb                     #SoC平台对应的libsophon安装文件
 37│   ├── sophon-soc-libsophon-dev_<x.y.z>_arm64.deb                 #SoC平台对应的libsophon安装文件
 38│   ├── system.tgz
 39│   └── tftp.tgz
 40├── sophon-mw_<date>_<hash>
 41│   ├── Multimedia FAQ.pdf
 42│   ├── Multimedia Technical Reference Manual.pdf
 43│   ├── Multimedia User Guide.pdf
 44│   ├── MULTIMEDIA使用手册.pdf
 45│   ├── MULTIMEDIA常见问题手册.pdf
 46│   ├── MULTIMEDIA开发参考手册.pdf
 47│   ├── release_version.txt
 48│   ├── sophon-mw_<x.y.z>_aarch64.tar.gz                          #arm64机器，其他LINUX系统对应的sophon-mw安装包
 49│   ├── sophon-mw_<x.y.z>_x86_64.tar.gz                           #x86_64机器，其他LINUX系统对应的sophon-mw安装包
 50│   ├── sophon-mw.MD5
 51│   ├── sophon-mw-soc_<x.y.z>_aarch64.tar.gz
 52│   ├── sophon-mw-soc-sophon-ffmpeg_<x.y.z>_arm64.deb             #SoC平台，ffmpeg运行时环境安装文件
 53│   ├── sophon-mw-soc-sophon-ffmpeg-dev_<x.y.z>_arm64.deb         #SoC平台，ffmpeg开发环境安装文件
 54│   ├── sophon-mw-soc-sophon-opencv_<x.y.z>_arm64.deb             #SoC平台，opencv运行时环境安装文件
 55│   ├── sophon-mw-soc-sophon-opencv-dev_<x.y.z>_arm64.deb         #SoC平台，opencv开发环境安装文件
 56│   ├── sophon-mw-soc-sophon-sample_<x.y.z>_arm64.deb             #SoC平台，多媒体程序示例文件
 57│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_amd64.deb                 #x86_64机器，Debian/Ubuntu系统对应的ffmpeg运行时环境安装文件
 58│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_arm64.deb                 #arm64机器，Debian/Ubuntu系统对应的ffmpeg运行时环境安装文件
 59│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_amd64.deb             #x86_64机器，Debian/Ubuntu系统对应的ffmpeg开发环境安装文件
 60│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_arm64.deb             #arm64机器，Debian/Ubuntu系统对应的ffmpeg开发环境安装文件
 61│   ├── sophon-mw-sophon-opencv_<x.y.z>_amd64.deb                 #x86_64机器，Debian/Ubuntu系统对应的opencv运行时环境安装文件
 62│   ├── sophon-mw-sophon-opencv_<x.y.z>_arm64.deb                 #arm64机器，Debian/Ubuntu系统对应的opencv运行时环境安装文件
 63│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_amd64.deb            #x86_64机器，CenterOS系统对应的opencv运行时环境安装文件
 64│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_arm64.deb            #arm64机器，CenterOS系统对应的opencv运行时环境安装文件
 65│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_amd64.deb        #x86_64机器，CenterOS系统对应的opencv开发环境安装文件
 66│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_arm64.deb        #arm64机器，CenterOS系统对应的opencv开发环境安装文件
 67│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_amd64.deb             #x86_64机器，Debian/Ubuntu系统对应的opencv开发环境安装文件
 68│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_arm64.deb             #arm64机器，Debian/Ubuntu系统对应的opencv开发环境安装文件
 69│   ├── sophon-mw-sophon-sample_<x.y.z>_amd64.deb                 #x86_64机器，多媒体程序示例文件
 70│   └── sophon-mw-sophon-sample_<x.y.z>_arm64.deb                 #arm64机器，多媒体程序示例文件
 71├── sophon-pipeline_<date>_<hash>
 72│   ├── release_version.txt
 73│   ├── sophon-pipeline.MD5
 74│   └── sophon-pipeline_v<x.y.z>_586366b_20221027.tar.gz          #基于pipeline的高性能推理框架
 75├── sophon-rpc_<date>_<hash>
 76│   ├── release_version.txt
 77│   ├── sophon-rpc_<x.y.z>_amd64.deb                              #x86_64机器，Debian/Ubuntu系统对应的sophon-rpc安装文件
 78│   ├── sophon-rpc_<x.y.z>_arm64.deb                              #arm64机器，Debian/Ubuntu系统对应的sophon-rpc安装文件
 79│   ├── sophon-rpc_<x.y.z>.tar.gz                                 #其他系统对应的sophon-rpc安装包
 80│   ├── sophon-rpc.MD5
 81│   └── sophon-rpc使用指南.pdf
 82├── sophon-sail_<date>_<hash>
 83│   ├── release_version.txt
 84│   ├── Sophon_Inference_zh.pdf
 85│   ├── sophon-sail_<x.y.z>.tar.gz                                #封装了BMLib、BMDecoder、BMCV、BMRuntime的高级接口库
 86│   └── sophon-sail.MD5
 87├── Sophon SDK文件清单.docx
 88├── tpu-kernel_<date>_<hash>
 89│   ├── release_version.txt
 90│   ├── tpu-kernel-1684x_v<x.y.z>-0d0e513e-221027.tar.gz          #自定义算子开发工具
 91│   └── tpu-kernel.MD5
 92├── tpu-mlir_<date>_<hash>
 93│   ├── release_version.txt
 94│   ├── tpu-mlir.MD5
 95│   └── tpu-mlir_v<x.y.z>-<hash>-<date>.tar.gz                    #编译器工具链
 96├── tpu-nntc_<date>_<hash>
 97│   ├── release_version.txt
 98│   ├── tpu-nntc.MD5
 99│   └── tpu-nntc_v<x.y.z>-<hash>-<date>.tar.gz                    #模型编译量化工具链
100└── tpu-perf_v<x.y.z>
101    ├── md5sum.txt
102    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_aarch64.whl
103    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_x86_64.whl
104    └── tpu-perf-<x.y.z>.tar.gz                                   #模型的性能分析和精度验证工具
```
### SDK主要模块

* **硬件驱动及运行时库 LIBSOPHON** ： 包含BMCV、BMRuntime、BMLib等库，用来驱动VPP、TPU等硬件，完成图像处理、张量运算、模型推理等操作。
* **多媒体库 SOPHON-MW** : 支持Sophon设备硬件加速的BM-OpenCV和BM-FFmpeg，支持RTSP流、GB28181流的解析，视频及图片的编解码。
* **模型编译量化工具链 TPU-NNTC** ： 支持Caffe、Tensorflow、Pytorch、MXNet、 Darknet、Paddle Paddle、ONNX等框架模型的模型转换；支持模型量化：原始模型 -> FP32 UModel -> INT8 UModel -> INT8 BModel， 同时提供 auto-cali 自动量化工具。
* **张量运算及图像处理库 BMCV** ： 色彩空间转换、尺度变换、仿射变换、投射变换、线性变换、画框、JPEG编码、BASE64编码、NMS、排序、特征匹配。
* **设备管理 BMLib** ： 基础接口：设备Handle的管理，内存管理、数据搬运、API的发送和同步、A53使能等
* **算丰AI加速库 SAIL** ： 支持Python/C++的高级接口，是对BMRuntime、BMCV、BMDecoder等底层库接口的封装。
* **自定义算子高级编程库 BMLang**：基于C++的面向Sophon TPU的高级编程库，与硬件信息解耦，无需了解硬件架构，使用张量数据（bmlang::Tensor）和计算操作（bmlang::Operator）编写代码，最后使用bmlang::compile或bmlang::compile_with_check来生成TPU可以运行的BModel；此外也支持使用BM168X中的arm cpu来实现TPU尚不支持的算子。
* **算法并行加速编程库 TPUKernel**：基于Sophon芯片底层原子操作接口的底层编程接口，需要熟悉硬件架构和指令集。
* **模型性能和精度验证工具 TPUPerf** : 可对模型进行性能分析和精度验证。

### 详细资料

* [获取 SDK（v23.05.01 版本）](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/zh/html/sdk_intro/3_claim_sdk.html)  
* [安装 SDK（v23.05.01 版本）](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/zh/html/sdk_intro/4_install.html#id12)
  
更多资料的，请参考：[https://developer.sophgo.com/site/index/document/all/all.html](https://developer.sophgo.com/site/index/document/all/all.html)    

[下载中心]: https://www.t-firefly.com/doc/download/220.html