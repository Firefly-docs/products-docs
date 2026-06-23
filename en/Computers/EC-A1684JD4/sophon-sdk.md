# Sophon SDK development
There are currently two sets of Sophon SDKs: Sophon SDK and SophonSDK3. Among them, Sophon SDK is the new version of the SDK, while SophonSDK3 is the old version.

The main differences between these two SDKs are as follows:


| Difference                      | Sophon SDK                                                                                                                                                                | SophonSDK3                                                                    | 
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |  --------------------------------- |
| Adaptor Chip                    | BM1684, BM1684X                                                                                                                                                           | BM1684, BM1684X                                                               | 
| Release Mode                    | Decouple each module and provide tar and deb packages respectively.                                                                                                       | Provides a unified tar development package that includes Compiler and Library | 
| Driver Installation             | Install the sophon-driver_x.y.z_amd64.deb package                                                                                                                         | Install using ‘install_driver_{target}.sh’                                  | 
| NNToolchain, Quantization-Tools | Provide tpu-nntc_vx.y.z- < hash >-< data > .tar.gz installation                                                                                                           | Included in SophonSDK3                                                        | 
| BMCV,BMRuntime, BMLib           | Provide sophon-libsophon_x.x.x_amd64.deb package, located at /opt/sophon/libsophon-current after installation                                                             | Included in SophonSDK3                                                        | 
| Multimedia Library              | Provide sophon-mw-x.x.x_amd64.deb package,located at /opt/sophon/sophon-ffmpeg-latest,/opt/sophon/sophon-opencv-latest,/opt/sophon/opencv-bmcpu-latest after installation | Included in SophonSDK3                                                        | 
| pipeline Routine                | Open source to github repository, sophon-pipeline                                                                                                                         | [https://github.com/sophon-ai-algo/sophon-inference](https://github.com/sophon-ai-algo/sophon-inference)                                                                              | 
| Algorithm Routine               | Open source to github repository (not released yet)                                                                                                                       |                                                                               | 

## SophonSDK3

SophonSDK is a deep learning SDK customized by Sophon Technology based on its self-developed AI chip. It covers model optimization and efficient runtime support required in the neural network inference stage. It provides easy-to-use and efficient development and deployment of deep learning applications. full-stack solution.

**Note: SophonSDK is used to develop applications directly on the PC, so users will not be involved in the development of the underlying hardware and operating system!**

### SDK introduction

#### SDK composition

SophonSDK consists of Compiler and Library:

- **Compiler** is responsible for offline compilation and optimization of the neural network model trained under the third-party deep learning framework, and generates the BModel required for the final runtime. Currently supports Caffe, Darknet, MXNet, ONNX, PyTorch, PaddlePaddle, TensorFlow, etc.

- **Library** provides BM-OpenCV, BM-FFmpeg, BMCV, TPURuntime, BMLib and other libraries to drive VPP, VPU, JPU, TPU and other hardware to complete video image encoding and decoding, image processing, tensor operations, models Inference and other operations for users to develop deep learning applications.

> Examples provides multiple examples in SoC and x86 environments for users to refer to in the development of deep learning applications. This part of the routine code is moved from the SDK to the open source github repository: [https://github.com/sophon-ai-algo/examples](https://github.com/sophon-ai-algo/examples)

![](../../../bm1684_img/sophonsdk_overview.png)

#### SDK development kit

SophonSDK provides two files, the specific files are shown in the following table:

| File name | Description |
| :------- | :--------- |
| [sophonsdk_v3.0.0_20220716.zip](https://sophon-file.sophon.cn/sophon-prod-s3/drive/22/07/18/11/sophonsdk_v3.0.0_20220716.zip) | sophonsdk x86 platform development package that supports cmodel, pcie, soc and arm_pcie modes |
| [x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip](https://sophon-file.sophon.cn/sophon-prod-s3/drive/22/07/19/10/x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip) | sophonsdk v3 compile docker for development |

- **SophonSDK3 Development Kit**: `sophonsdk_v<x.y.z>.zip`, where x.y.z is the version number. SophonSDK will be mounted inside the Docker container in the form of directory mapping for users to use. The SDK is divided into Windows version and Linux version. For how to use the Windows version of the SDK, please refer to the user documentation under the SDK package.

- **Docker for x86 based Ubuntu development**: used to provide a unified Docker basic development image to avoid hidden problems caused by inconsistent environments, especially for beginners, it is strongly recommended to use Docker images as a development environment for model conversion and Algorithm migration works.

**Notice:**

**SophonSDK is a development kit that supports multiple platforms. In order to adapt to the needs of different hardware platforms and compilers, developers need to run a simple installation script after decompressing the SDK. This installation script will adjust and install the appropriate program version according to the user's current host environment, such as ABI (application Program Binary Interface) version. Therefore, it is not recommended to copy the decompressed folder to other machines for use. If there is a need for copying, please copy the original compressed package of the SDK, and then decompress and install it again.**
- **For example, the default ABI format under CentOS and Ubuntu is different and cannot be used universally. When the copied file is executed, it will report an error that the symbol cannot be found;**
- **When copying the decompressed files using an NTFS formatted mobile storage device, especially when copying files between Windows and Linux, it is very likely to cause file link or so file damage.**

#### SDK structure

The root directory after the SDK is decompressed: on the host, it is the path to store the files specified when decompressing the files; when in the docker container, if the container creation script under the SDK is not modified, the path defaults to `/workspace`.

The decompressed SDK structure is as follows:

```shell
SophonSDK3

├── bin                    # Related tools for each platform
│   ├── arm                # soc platform, corresponding to SE5/SM5 devices
│   ├── arm_pcie           # ARM instruction set CPU server host
│   ├── fib.bin            # 
│   ├── firmware           # MCU firmware in BM1684
│   ├── loongarch64        # The server host of the loongarch instruction set CPU
│   ├── mips64             # MIPS instruction set CPU server host
│   ├── ramboot_rootfs.itb # 
│   ├── spi_flash.bin      #
│   ├── sw64               # SW64 instruction set CPU server host
│   └── x86                # intel x86 host
├── bmlang                 # High-level programming interface for Sophon TPU
├── bmnet                  # Compiler tool executable or pip install whl package or file
│   ├── bmcompiler         # compiler
│   ├── bmnetc             # Caffe Compiler
│   ├── bmnetd             # Darknet Compiler
│   ├── bmnetm             # MXNet Compiler
│   ├── bmneto             # ONNX Compiler
│   ├── bmnetp             # Pytorch Compiler
│   ├── bmnett             # TensorFlow Compiler
│   ├── bmnetu             # int8 Umodel compiler
│   ├── bmpaddle           # PaddlePaddle Compiler 
│   ├── bmprofile          # Performance analysis tool
│   ├── bmtflite           # TFLite Compiler
│   ├── bmusercpu          # Use source files related to A53 arm cpu
│   ├── calibration        # Quantification tool ufw installation package
│   └── debugtools         # Debug tool source code
├── driver                 # PCIE card device driver source code
├── include                # Runtime library header file for secondary development, one folder for each module
├── lib                    # Runtime and third-party libraries for runtime and secondary development
├── release_version.txt    # SDK release version number
├── res                    # test image
├── scripts                # Common scripts: driver installation, software library installation, environment variable setting
├── docker_run_sophonsdk.sh # Docker startup script
└── test                   # Test script, users do not need to pay attention
```

#### SDK main modules

The main modules are as follows:

- **Codec library Multimedia**: Supports BM-OpenCV and BM-FFmpeg for Sophon device hardware acceleration, supports RTSP stream, GB28181 stream parsing, video and picture encoding and decoding.
- **Tensor operation and image processing library BMCV**: color space conversion, scale transformation, affine transformation, projection transformation, linear transformation, picture frame, JPEG encoding, BASE64 encoding, NMS, sorting, feature matching.
- **Device management BMLib**: basic interface: device handle management, memory management, data handling, API transmission and synchronization, A53 enable, etc.
- **Model conversion toolchain NNToolchain**: supports framework models such as Caffe, Tensorflow, Pytorch, MXNet, Darknet, Paddle Paddle, ONNX; provides BMRuntime for model inference; provides BMProfile tool for performance analysis.
- **Model quantization tools Quantization-Tools**: original model -> FP32 UModel -> INT8 UModel -> INT8 BModel; provides auto-calib automatic quantization tools.
- **Suanfeng AI Acceleration Library SAIL**: It supports the high-level interface of Python/C++, and is the encapsulation of low-level library interfaces such as BMRuntime, BMCV, and BMDecoder.
- **Custom operator advanced programming library BMLang**: C++-based advanced programming library for Sophon TPU, decoupled from hardware information, no need to understand hardware architecture, using tensor data (bmlang::Tensor) and computing operations ( bmlang::Operator) to write code, and finally use bmlang::compile or bmlang::compile_with_check to generate a BModel that can be run by the TPU; in addition, it also supports the use of arm cpu in BM168X to implement operators that are not yet supported by the TPU.
- **Algorithm Parallel Acceleration Programming Library OKKernel (TPUKernel)**: The underlying programming interface based on the underlying atomic operation interface of the Sophon chip, requires familiarity with the hardware architecture and instruction set.

#### Environment configuration

Due to space limitations, the configuration in the Ubuntu environment is mainly introduced here.

Docker needs to be installed and the development user account has docker permissions. For installing the Docker environment, please refer to "[Docker manual](https://docs.docker.com/engine/install)".

Download [sophonsdk](https://developer.sophon.ai/site/index/material/all/all.html) and [docker development image](https://developer.sophon.ai/site/index/material/11/all.html) and unzip:

```shell
unzip sophonsdk_v3.0.0_20220716.zip
unzip x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker.zip
cd sophonsdk_v3.0.0_20220716
tar xzvf sophonsdk_v3.0.0.tar.gz
```

Load the docker image:

```shell
cd x86_sophonsdk3_ubuntu18.04_py37_dev_22.06_docker
docker load -i x86_sophonsdk3_ubuntu18.04_py37_dev_22.06.docker
```

Create a docker container and enter docker:

```shell
cd sophonsdk_v3.0.0
# If you do not perform the configuration operation about the docker command without root execution, you need to add sudo before the command
./docker_run_sophonsdk.sh
```

After execution, you can see that you have entered the docker's `workspace` directory:

```shell
root@t-firefly:/workspace#
```

Working in CModel mode:

```shell
cd /workspace/scripts/
./install_lib.sh nntc
# Set environment variables, note that this command is only valid for the current terminal, and re-entry needs to be executed again
source envsetup_cmodel.sh
```

Verify that the cross-compilation toolchain in the development environment is configured successfully:

```shell
which aarch64-linux-gnu-g++
# terminal output
# /data/release/toolchains/gcc/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin//aarch64-linux-gnu-g++
```

If the terminal outputs the path of aarch64 compilation, it means that the cross-compilation toolchain is correct and the development environment can be used normally.

#### Details

For more detailed information, please refer to: [https://developer.sophon.ai/site/index/document/all/all.html](https://developer.sophon.ai/site/index/document/all/all.html)

## Sophon SDK

Sophon SDK is a customized deep learning SDK based on its self-developed AI chip, covering model optimization and efficient operation support required for the inference phase of neural networks, providing an easy-to-use and efficient full-stack solution for deep learning application development and deployment.

SophonSDK is compatible with both the 3rd generation BM1684 chip and the 4th generation BM1684X chip.

### Components of Sophon SDK

#### Basic Toolkit

The basic toolkit includes:

* Tpu-nntc is responsible for the offline compilation and optimization of the neural network model trained under the third-party deep learning framework to generate the final run-time BModel. Caffe, Darknet, MXNet, ONNX, PyTorch, PaddlePaddle, TensorFlow and so on are supported.
* Libsophon provides BMCV, BMRuntime, BMLib and other libraries, which are used to drive VPP, TPU and other hardware to complete image processing, tensor operation, model inference and other operations for users to develop deep learning applications.
* Sophon-mw encapsulates BM-OpenCV, BM-FFmpeg and other libraries, which are used to drive VPU, JPU and other hardware, and support the parsing of RTSP stream and GB28181 stream, video image codec acceleration, etc., for users to carry out deep learning application development.
* Sophon-sail provides advanced interfaces supporting Python/C++, which encapsulates the underlying library interfaces such as BMRuntime, BMCV, BMDecoder, BMLib, etc., for users to carry out deep learning application development.

#### Advanced Toolkit

The Advanced Toolkit includes:

* Tpu-mlir provides a complete tool chain for the TPU compiler project, which can transform pre-trained neural networks in different frameworks into binary BModel that can run efficiently on the Sophon TPU. Frameworks that are currently directly supported include tflite, onnx, and Caffe.
* Tpu-perf provides a complete set of toolkits for model performance and accuracy verification.
* Tpu-kernel is the underlying development interface to the chip, allowing both dedicated instructions to accelerate deep learning business logic and generic instructions to accelerate various custom algorithms.

![../_images/new_SophonSDK.png](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/en/html/_images/new_SophonSDK.png)

### SDK Directory Structure

SophonSDK provides eleven folder modules, as shown in the following table:

| Folder Name     | Remarks                                                                                                                         |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| libsophon       | Libraries supporting hardware acceleration of sophon devices, such as image processing, tensor operation, model inference, etc. |
| sophon-mw       | Multimedia Library supporting hardware acceleration of sophon Devices                                                           |
| tpu-kernel      | Underlying development interface                                                                                                |
| tpu-nntc        | TPU compiler tool chain                                                                                                         |
| tpu-mlir        | TPU compiler tool chain                                                                                                         |
| tpu-perf        | Model performance and accuracy verification toolkit                                                                             |
| sophon-pipeline | A pipeline-based high-performance inference framework                                                                           |
| sophon-img      | SoC Mode Installation Kit                                                                                                       |
| sophon-demo     | Comprehensive routines for single models or scenarios                                                                           |
| sophon-sail     | Interface library that encapsulates the underlying interface into C++/Python API                                                |
| sophon-rpc      | Offload some computing tasks on PCIe card product to ARM CPU on the card                                                        |
   
### Unzipped SDK Directory

The structure of the unpacked SDK files is as follows:
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
 10│   ├── libsophon_<x.y.z>_aarch64.tar.gz                           #Arm64 machine, libsophon installar for other LINUX systems
 11│   ├── libsophon_<x.y.z>_x86_64.tar.gz                            #X86x64 machine, libsophon installar for other LINUX systems
 12│   ├── libsophon_dockerfile
 13│   ├── libsophon.MD5
 14│   ├── LIBSOPHON_User_Guide.pdf
 15│   ├── LIBSOPHON使用手册.pdf
 16│   ├── release_version.txt
 17│   ├── sophon-driver_<x.y.z>_amd64.deb                            #x86_64 machine，PCIe card driver installation file corresponding to Debian/Ubuntu system
 18│   ├── sophon-driver_<x.y.z>_arm64.deb                            #arm64 machine，PCIe card driver installation file corresponding to Debian/Ubuntu system
 19│   ├── sophon-libsophon_<x.y.z>_amd64.deb                         #x86_64 machine，libsophon runtime environment installation file corresponding to Debian/Ubuntu system
 20│   ├── sophon-libsophon_<x.y.z>_arm64.deb                         #arm64 machine，libsophon runtime environment installation file corresponding to Debian/Ubuntu system
 21│   ├── sophon-libsophon-dev_<x.y.z>_amd64.deb                     #x86_64 machine，libsophon runtime environment installation file corresponding to Debian/Ubuntu system
 22│   └── sophon-libsophon-dev_<x.y.z>_arm64.deb                     #arm64 machine，libsophon runtime environment installation file corresponding to Debian/Ubuntu system
 23├── sophon-demo_<date>_<hash>
 24│   ├── release_version.txt
 25│   ├── sophon-demo.MD5
 26│   └── sophon-demo_v<x.y.z>_b909566_20221027.tar.gz               #Comprehensive routines for a single model or scenario
 27├── sophon-img_<date>_<hash>
 28│   ├── bsp-debs
 29│   ├── bsp_update.tgz
 30│   ├── libsophon_soc_<x.y.z>_aarch64.tar.gz                       #Files required for cross-compilation
 31│   ├── release_version.txt
 32│   ├── sdcard.tgz                                                 #SoC card brush package
 33│   ├── SOPHON_BSP_Technical_Reference_Manual.pdf
 34│   ├── SOPHON BSP开发参考手册.pdf
 35│   ├── sophon-img.MD5
 36│   ├── sophon-soc-libsophon_<x.y.z>_arm64.deb                     #Libsophon installation files corresponding to SoC platform
 37│   ├── sophon-soc-libsophon-dev_<x.y.z>_arm64.deb                 #Libsophon installation files corresponding to SoC platform
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
 48│   ├── sophon-mw_<x.y.z>_aarch64.tar.gz                          #Arm64 machine, sophon-mw installar for other LINUX systems
 49│   ├── sophon-mw_<x.y.z>_x86_64.tar.gz                           #Arm64 machine, sophon-mw installar for other LINUX systems
 50│   ├── sophon-mw.MD5
 51│   ├── sophon-mw-soc_<x.y.z>_aarch64.tar.gz
 52│   ├── sophon-mw-soc-sophon-ffmpeg_<x.y.z>_arm64.deb             #SoC platform, ffmpeg runtime environment installation files
 53│   ├── sophon-mw-soc-sophon-ffmpeg-dev_<x.y.z>_arm64.deb         #SoC platform, ffmpeg development environment installation files
 54│   ├── sophon-mw-soc-sophon-opencv_<x.y.z>_arm64.deb             #SoC platform, opencv runtime environment installation files
 55│   ├── sophon-mw-soc-sophon-opencv-dev_<x.y.z>_arm64.deb         #SoC platform, opencv development environment installation files
 56│   ├── sophon-mw-soc-sophon-sample_<x.y.z>_arm64.deb             #SoC platform, multimedia program sample file
 57│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_amd64.deb                 #X86x64 machine, installation files for ffmpeg runtime environment corresponding to Debian/Ubuntu system
 58│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_arm64.deb                 #arm64 machine, installation files for ffmpeg runtime environment corresponding to Debian/Ubuntu system
 59│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_amd64.deb             #x86_64 machine, ffmpeg development environment installation file corresponding to Debian/Ubuntu system
 60│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_arm64.deb             #arm64 machine, ffmpeg development environment installation file corresponding to Debian/Ubuntu system
 61│   ├── sophon-mw-sophon-opencv_<x.y.z>_amd64.deb                 #x86_64 machine, opencv runtime environment installation file corresponding to Debian/Ubuntu system
 62│   ├── sophon-mw-sophon-opencv_<x.y.z>_arm64.deb                 #arm64 machine, opencv runtime environment installation file corresponding to Debian/Ubuntu system
 63│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_amd64.deb            #X86x64 machine, installation files for opencv runtime environment corresponding to CenterOS system
 64│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_arm64.deb            #arm64 machine, opencv runtime environment installation file corresponding to CenterOS system
 65│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_amd64.deb        #x86x64 machine, opencv development environment installation file corresponding to CenterOS system
 66│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_arm64.deb        #arm64 machine, opencv development environment installation file corresponding to CenterOS system
 67│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_amd64.deb             #x86x64 machine, opencv development environment installation file corresponding to Debian/Ubuntu system
 68│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_arm64.deb             #arm64 machine, opencv development environment installation file corresponding to Debian/Ubuntu system
 69│   ├── sophon-mw-sophon-sample_<x.y.z>_amd64.deb                 #x86x64 machine, sample files for multimedia programs
 70│   └── sophon-mw-sophon-sample_<x.y.z>_arm64.deb                 #arm64 machine, sample files of multimedia programs
 71├── sophon-pipeline_<date>_<hash>
 72│   ├── release_version.txt
 73│   ├── sophon-pipeline.MD5
 74│   └── sophon-pipeline_v<x.y.z>_586366b_20221027.tar.gz          #High performance inference framework based on pipeline
 75├── sophon-rpc_<date>_<hash>
 76│   ├── release_version.txt
 77│   ├── sophon-rpc_<x.y.z>_amd64.deb                              #x86x64 machine, sophon-rpc installation file corresponding to Debian/Ubuntu system
 78│   ├── sophon-rpc_<x.y.z>_arm64.deb                              #arm64 machine, sophon-rpc installation file corresponding to Debian/Ubuntu system
 79│   ├── sophon-rpc_<x.y.z>.tar.gz                                 #Sophon-rpc installation packages corresponding to other systems
 80│   ├── sophon-rpc.MD5
 81│   └── sophon-rpc使用指南.pdf
 82├── sophon-sail_<date>_<hash>
 83│   ├── release_version.txt
 84│   ├── Sophon_Inference_zh.pdf
 85│   ├── sophon-sail_<x.y.z>.tar.gz                                #Advanced interface library encapsulating BMLib, BMDecoder, BMCV, BMRuntime
 86│   └── sophon-sail.MD5
 87├── Sophon SDK文件清单.docx
 88├── tpu-kernel_<date>_<hash>
 89│   ├── release_version.txt
 90│   ├── tpu-kernel-1684x_v<x.y.z>-0d0e513e-221027.tar.gz          #Custom operator development tool
 91│   └── tpu-kernel.MD5
 92├── tpu-mlir_<date>_<hash>
 93│   ├── release_version.txt
 94│   ├── tpu-mlir.MD5
 95│   └── tpu-mlir_v<x.y.z>-<hash>-<date>.tar.gz                    #Compiler tool chain
 96├── tpu-nntc_<date>_<hash>
 97│   ├── release_version.txt
 98│   ├── tpu-nntc.MD5
 99│   └── tpu-nntc_v<x.y.z>-<hash>-<date>.tar.gz                    #Model compilation quantization tool chain
100└── tpu-perf_v<x.y.z>
101    ├── md5sum.txt
102    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_aarch64.whl
103    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_x86_64.whl
104    └── tpu-perf-<x.y.z>.tar.gz                                   #Tools for performance analysis and accuracy verification of the model

```

### SDK Main Modules

* **Hardware Driver and Runtime Library LIBSOPHON** : contains libraries such as BMCV, BMRuntime, BMLib, etc., which are used to drive hardware such as VPP, TPU, etc. toperform operations such as image processing, tensor operations, model inference, etc
* **Multimedia library SOPHON-MW** : supports BM-OpenCV and BM-FFmpeg for hardware accelerate of Sophon devices, parsing of RTSP streams, GB28181 streams, video and image encoding and decoding
* **Model Compilation Quantization Tool Chain TPU-NNTC** : supports model conversion of Caffe,Tensorflow,Pytorch,MXNet,Darknet, Paddle Paddle, ONNX and other framework models; supports model quantization: original model -> FP32 UModel -> INT8 UModel -> INT8 BModel, and also provides auto-cali automatic quantization tool.
* **Tensor Operation and Image Processing Library BMCV** : colour space conversion, scale transformation, affine transformation, projection transformation, linear transformation, frame, JPEG coding, BASE64 coding, NMS, sorting, feature matching
* **Equipment Management BMLib** : basic interfaces: management of device Handle, memory management, data handling, API sending and synchronisation, A53 enable, etc
* **SAIL, the Arithmetic AI acceleration library** : supports high-level interfaces to Python/C++ and is wrapper for underlying library interfaces such as BMRuntime、BMCV、BMDecoder
* **Custom Operator Advanced Programming Library BMLang**: A C++ based high-level programming library for Sophon TPUs, decoupled from hardware information, no knowledge of hardware architecture is required, code is written using tensor data (bmlang::Tensor) and computation operations (bm- lang::Operator), and finally bmlang::compile or bmlang::compile_with_check is used to generate a BModel that the TPU can run; it also supports the use of the arm cpu in BM168X to implement arithmetic that is not yet supported by the TPU.
* **Algorithm Parallel Accelerated Programming Library TPUKernel**: underlying programming interface based on the Sophon chip’s underlying atomic operations interface, requiring familiarity with hardware architecture and instruction set
* **Model Performance and Accuracy Verification Tool TPUPerf**: enables performance analysis and accuracy validation of models

### Details
* [Get SDK（v23.05.01 version）](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/en/html/sdk_intro/3_claim_sdk.html)  
* [Install SDK（v23.05.01 version）](https://doc.sophgo.com/sdk-docs/v23.05.01/docs_latest_release/docs/SophonSDK_doc/en/html/sdk_intro/4_install.html)

For additional resources, please visit [here](https://developer.sophon.ai/site/index/material/all/all.html).