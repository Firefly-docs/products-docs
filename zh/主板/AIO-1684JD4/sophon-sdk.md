# Sophon SDK 开发

目前存在两套 Sophon SDK，其中 **SophonSDK3** 是属于旧版本的 SDK，而 **Sophon SDK** 属于新版本的 SDK。
两套 SDK 的主要区别在于：  

| 区别                            | Sophon SDK                                                                                                                                     | SophonSDK3                                    |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |----------------------------------------------------------------------------------------------------------------------------------------------- |
| 适配芯片                        | BM1684、BM1684X                                                                                                                                | BM1684、BM1684X                               |
| 发布方式                        | 将各个模块解耦，分别提供tar和deb包。                                                                                                           | 提供统一的tar开发包， 包含Compiler和Library。 | 
| 驱动安装                        | 安装sophon-driver_x.y.z_amd64.deb包                                                                                                            | 使用install_driver_{target}.sh安装            
| NNToolchain、Quantization-Tools | 提供tpu-nntc_vx.y.z-<hash>-<data>.tar.gz安装使用                                                                                               | 包含于SophonSDK3                              | 
| BMCV、BMRuntime、BMLib          | 提供sophon-libsophon_x.x.x_amd64.deb包，安装后位于/opt/sophon/libsophon-current                                                                | 包含于SophonSDK3                              | 
| 多媒体库                        | 提供sophon-mw-x.x.x_amd64.deb包，安装后位于/opt/sophon/sophon-ffmpeg-latest、/opt/sophon/sophon-opencv-latest、/opt/sophon/opencv-bmcpu-latest | 包含于SophonSDK3                              | 
| pipeline例程                    | 开源至github仓库，sophon-pipeline                                                                                                              | [https://github.com/sophon-ai-algo/sophon-inference](https://github.com/sophon-ai-algo/sophon-inference)                                              | 
| 算法例程                        | 开源至github仓库（暂未发布）                                                                                                                   | [https://github.com/sophon-ai-algo/examples/tree/2.7.0](https://github.com/sophon-ai-algo/examples/tree/2.7.0)                                              | 

建议优先使用 `Ubuntu 20.0`的固件以及新版本的 SDK 去做开发。

##  SophonSDK3

SophonSDK 是算能科技基于其自主研发的 AI 芯片所定制的深度学习 SDK，涵盖了神经网络推理阶段所需的模型优化、高效运行时支持等能力，为深度学习应用开发和部署提供易用、高效的全栈式解决方案。

**注意：SophonSDK 是用于在 PC 上直接开发应用程序的，因此用户不会涉及到底层硬件与操作系统开发！**



### SDK 组成

SophonSDK 由 Compiler 和 Library 组成：

- **Compiler** 负责对第三方深度学习框架下训练得到的神经网络模型进行离线编译和优化，生成最终运行时需要的 BModel。目前支持 Caffe、Darknet、MXNet、ONNX、PyTorch、PaddlePaddle、TensorFlow 等。

- **Library** 提供 BM-OpenCV、BM-FFmpeg、BMCV、TPURuntime、BMLib 等库，用来驱动 VPP、VPU、JPU、TPU 等硬件，完成视频图像编解码、图像处理、张量运算、模型推理等操作，供用户进行深度学习应用开发。

> Examples 提供了 SoC 和 x86 环境下的多个例子，供用户在深度学习应用开发过程中参考。这部分例程代码从 SDK 中移至 github 仓库开源：[https://github.com/sophon-ai-algo/examples](https://github.com/sophon-ai-algo/examples)

![](../../../bm1684_img/sophonsdk_overview.png)

### SDK 开发包

SophonSDK 提供了两个文件，具体文件如下表所示：

| 文件名 | 备注 |
| :------- | :--------- |
| [sophonsdk_v3.0.0_20220716.zip](https://sophon-file.sophon.cn/sophon-prod-s3/drive/22/07/18/11/sophonsdk_v3.0.0_20220716.zip) | sophonsdk x86 平台开发包，支持 cmodel，pcie，soc 和 arm_pcie 模式 |
| [x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip](https://sophon-file.sophon.cn/sophon-prod-s3/drive/22/07/19/10/x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip) | sophonsdk v3 编译开发用的 docker |

- **SophonSDK3 开发包**：`sophonsdk_v<x.y.z>.zip`，其中 x.y.z 为版本号。SophonSDK 将以目录映射的方式挂载到 Docker 容器内部供用户使用。SDK 分 Windows 版本和 Linux 版本。Windows 版本的 SDK 的使用方式请参见 SDK 包下的用户文档。

- **基于 x86 的 Ubuntu 开发 Docker**：用于提供统一的 Docker 基础开发镜像以规避环境不一致带来的隐性问题，特别是对于初级用户，强烈建议使用 Docker 镜像作为开发环境进行模型转换和算法迁移工作。

**注意：**

**SophonSDK 是一个支持多种平台的开发包。为了适应不同硬件平台和编译器的需求，开发者拿到 SDK 解压缩后，需要运行一个简单的安装脚本，此安装脚本会根据用户当前的主机环境调整安装适配的程序版本，如 ABI（应用程序二进制接口）版本。因此，不建议拷贝解压缩后的文件夹到其他机器使用，确有拷贝需求时，请拷贝 SDK 原始压缩包，然后重新解压缩再安装。**
- **比如 CentOS 和 Ubuntu 下默认的 ABI 格式不一样，无法通用，拷贝后的文件执行时会报找不到符号的错误；**
- **当使用 NTFS 格式的移动存储设备拷贝解压后的文件时，特别是在 Windows 下和 Linux 之间拷贝文件时，极有可能造成文件链接或 so 文件损坏。**


### SDK 文件结构

SDK 解压后的根目录：在宿主机上时，它是解压文件时指定的存放文件的路径；在 docker 容器中时，若没有修改 SDK 下的创建容器脚本，该路径默认为`/workspace`。

解压后的 SDK 文件结构如下：

```shell
SophonSDK3

├── bin                    # 各个平台的相关工具
│   ├── arm                # soc平台，对应SE5/SM5设备
│   ├── arm_pcie           # ARM指令集CPU的服务器主机
│   ├── fib.bin            # 
│   ├── firmware           # BM1684中MCU的固件
│   ├── loongarch64        # loongarch指令集CPU的服务器主机
│   ├── mips64             # MIPS指令集CPU的服务器主机
│   ├── ramboot_rootfs.itb # 
│   ├── spi_flash.bin      #
│   ├── sw64               # SW64指令集CPU的服务器主机
│   └── x86                # intel x86主机
├── bmlang                 # 面向Sophon TPU的高级编程接口
├── bmnet                  # Compiler工具的可执行文件或pip安装whl包或文件
│   ├── bmcompiler         # compiler
│   ├── bmnetc             # Caffe Compiler
│   ├── bmnetd             # Darknet Compiler
│   ├── bmnetm             # MXNet Compiler
│   ├── bmneto             # ONNX Compiler
│   ├── bmnetp             # Pytorch Compiler
│   ├── bmnett             # TensorFlow Compiler
│   ├── bmnetu             # int8 Umodel compiler
│   ├── bmpaddle           # PaddlePaddle Compiler 
│   ├── bmprofile          # 性能分析工具
│   ├── bmtflite           # TFLite Compiler
│   ├── bmusercpu          # 使用A53 arm cpu相关的源文件
│   ├── calibration        # 量化工具ufw安装包
│   └── debugtools         # 调试工具源码
├── driver                 # PCIE卡设备驱动源码
├── include                # 运行库头文件，供二次开发使用，每个模块一个文件夹
├── lib                    # 运行库及第三方库，供运行时和二次开发使用
├── release_version.txt    # SDK发布版本号
├── res                    # 测试图片
├── scripts                # 常用脚本：驱动安装、软件库安装、环境变量设置
├── docker_run_sophonsdk.sh # Docker启动脚本
└── test                   # 测试脚本，用户无需关注
```

### SDK 主要模块

主要模块如下：

- **编解码库 Multimedia**：支持 Sophon 设备硬件加速的 BM-OpenCV 和 BM-FFmpeg，支持 RTSP流、GB28181 流的解析、视频及图片的编解码。
- **张量运算及图像处理库 BMCV**：色彩空间转换、尺度变换、仿射变换、投射变换、线性变换、画框、JPEG 编码、BASE64 编码、NMS、排序、特征匹配。
- **设备管理 BMLib**：基础接口：设备 Handle 的管理，内存管理、数据搬运、API 的发送和同步、A53 使能等。
- **模型转换工具链 NNToolchain**：支持 Caffe、Tensorflow、Pytorch、MXNet、 Darknet、Paddle Paddle、ONNX 等框架模型；提供 BMRuntime 用于模型推理；提供 BMProfile 工具用于性能分析。
- **模型量化工具 Quantization-Tools**：原始模型 -> FP32 UModel -> INT8 UModel -> INT8 BModel；提供 auto-calib 自动量化工具。
- **算丰 AI 加速库 SAIL**：支持 Python/C++ 的高级接口，是对 BMRuntime、BMCV、BMDecoder 等底层库接口的封装。
- **自定义算子高级编程库 BMLang**：基于 C++ 的面向 Sophon TPU 的高级编程库，与硬件信息解耦，无需了解硬件架构，使用张量数据（bmlang::Tensor）和计算操作（bmlang::Operator）编写代码，最后使用 bmlang::compile 或 bmlang::compile_with_check 来生成 TPU 可以运行的 BModel；此外也支持使用 BM168X 中的 arm cpu 来实现 TPU 尚不支持的算子。
- **算法并行加速编程库 OKKernel（TPUKernel）**：基于 Sophon 芯片底层原子操作接口的底层编程接口，需要熟悉硬件架构和指令集。

### 环境配置

限于篇幅，这里主要介绍 Ubuntu 环境下的配置。

需要安装好 Docker 且开发用户帐号有 docker 权限，关于安装 Docker 环境，可参考《[Docker 官方教程](https://docs.docker.com/engine/install)》。

下载 [sophonsdk](https://developer.sophgo.com/site/index/material/17/45.html) 与 [docker 开发镜像](https://developer.sophgo.com/site/index/material/11/all.html)并解压：

```shell
unzip sophonsdk_v3.0.0_20220716.zip
unzip x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip
cd sophonsdk_v3.0.0_20220716
tar xzvf sophonsdk_v3.0.0.tar.gz
```

加载 docker 镜像：

```shell
cd x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker
docker load -i x86_sophonsdk3_ubuntu18.04_py37_dev_22.06.docker
```

创建 docker 容器进入 docker：

```shell
cd sophonsdk_v3.0.0
# 若没有执行关于docker命令免root执行的配置操作，需在命令前添加sudo
./docker_run_sophonsdk.sh
```

执行后可以看到进入到 docker 的`workspace`目录：

```shell
root@t-firefly:/workspace#
```

工作在 CModel 模式：

```shell
cd /workspace/scripts/
./install_lib.sh nntc
# 设置环境变量，注意此命令只对当前终端有效，重新进入需要重新执行
source envsetup_cmodel.sh
```

验证开发环境中的交叉编译工具链是否配置成功：

```shell
which aarch64-linux-gnu-g++
# 终端输出内容
# /data/release/toolchains/gcc/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin//aarch64-linux-gnu-g++
```

如果终端输出了 aarch64 编译的路径，则说明交叉编译工具链正确，开发环境是可以正常使用的。

### 详细资料

更多详细资料可参考：[https://developer.sophgo.com/site/index/document/all/all.html](https://developer.sophgo.com/site/index/document/all/all.html)

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

[下载中心]: https://www.t-firefly.com/doc/download/179.html