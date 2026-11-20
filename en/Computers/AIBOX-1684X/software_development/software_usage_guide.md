# Software Usage Guide

<Callout title="Note" type="info">This page collects the software usage content of AIBOX-1684X: Sophon SDK development, memory management, remote desktop, and Qt development. Different computers support different features; please refer to the SDK or firmware of your actual device.</Callout>


## Sophon SDK Development [step]


### Sophon SDK [step]
Sophon SDK is a customized deep learning SDK based on its self-developed AI chip, covering model optimization and efficient operation support required for the inference phase of neural networks, providing an easy-to-use and efficient full-stack solution for deep learning application development and deployment.

SophonSDK is compatible with both the 3rd generation BM1684 chip and the 4th generation BM1684X chip.

#### Components of Sophon SDK [step]
##### Basic Toolkit

The basic toolkit includes:

* Tpu-nntc is responsible for the offline compilation and optimization of the neural network model trained under the third-party deep learning framework to generate the final run-time BModel. Caffe, Darknet, MXNet, ONNX, PyTorch, PaddlePaddle, TensorFlow and so on are supported.
* Libsophon provides BMCV, BMRuntime, BMLib and other libraries, which are used to drive VPP, TPU and other hardware to complete image processing, tensor operation, model inference and other operations for users to develop deep learning applications.
* Sophon-mw encapsulates BM-OpenCV, BM-FFmpeg and other libraries, which are used to drive VPU, JPU and other hardware, and support the parsing of RTSP stream and GB28181 stream, video image codec acceleration, etc., for users to carry out deep learning application development.
* Sophon-sail provides advanced interfaces supporting Python/C++, which encapsulates the underlying library interfaces such as BMRuntime, BMCV, BMDecoder, BMLib, etc., for users to carry out deep learning application development.

##### Advanced Toolkit

The Advanced Toolkit includes:

* Tpu-mlir provides a complete tool chain for the TPU compiler project, which can transform pre-trained neural networks in different frameworks into binary BModel that can run efficiently on the Sophon TPU. Frameworks that are currently directly supported include tflite, onnx, and Caffe.
* Tpu-perf provides a complete set of toolkits for model performance and accuracy verification.
* Tpu-kernel is the underlying development interface to the chip, allowing both dedicated instructions to accelerate deep learning business logic and generic instructions to accelerate various custom algorithms.

<center>

<img alt="" src="../../../../bm1684_img/new_SophonSDK.en.png" width="700">
</center>

#### SDK Directory Structure [step]
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
   
#### Unzipped SDK Directory [step]
The structure of the unpacked SDK files is as follows.

Note: Some of the manuals below have Chinese file names (for example, `BMCV开发参考手册.pdf`). They are the original Chinese documents bundled with the SDK, so their file names are kept as-is.

```
SophonSDK

├── libsophon_<date>_<hash>
│   ├── BMCV_Technical_Reference_Manual.pdf
│   ├── BMCV开发参考手册.pdf
│   ├── BMLib_Technical_Reference_Manual.pdf
│   ├── BMLIB开发参考手册.pdf
│   ├── BMRuntime Technical Reference Manual.pdf
│   ├── BMRUNTIME开发参考手册.pdf
│   ├── libsophon_<x.y.z>_aarch64.tar.gz                           #Arm64 machine, libsophon installar for other LINUX systems
│   ├── libsophon_<x.y.z>_x86_64.tar.gz                            #X86x64 machine, libsophon installar for other LINUX systems
│   ├── libsophon_dockerfile
│   ├── libsophon.MD5
│   ├── LIBSOPHON_User_Guide.pdf
│   ├── LIBSOPHON使用手册.pdf
│   ├── release_version.txt
│   ├── sophon-driver_<x.y.z>_amd64.deb                            #x86_64 machine,PCIe card driver installation file corresponding to Debian/Ubuntu system
│   ├── sophon-driver_<x.y.z>_arm64.deb                            #arm64 machine,PCIe card driver installation file corresponding to Debian/Ubuntu system
│   ├── sophon-libsophon_<x.y.z>_amd64.deb                         #x86_64 machine,libsophon runtime environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-libsophon_<x.y.z>_arm64.deb                         #arm64 machine,libsophon runtime environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-libsophon-dev_<x.y.z>_amd64.deb                     #x86_64 machine,libsophon runtime environment installation file corresponding to Debian/Ubuntu system
│   └── sophon-libsophon-dev_<x.y.z>_arm64.deb                     #arm64 machine,libsophon runtime environment installation file corresponding to Debian/Ubuntu system
├── sophon-demo_<date>_<hash>
│   ├── release_version.txt
│   ├── sophon-demo.MD5
│   └── sophon-demo_v<x.y.z>_b909566_20221027.tar.gz               #Comprehensive routines for a single model or scenario
├── sophon-img_<date>_<hash>
│   ├── bsp-debs
│   ├── bsp_update.tgz
│   ├── libsophon_soc_<x.y.z>_aarch64.tar.gz                       #Files required for cross-compilation
│   ├── release_version.txt
│   ├── sdcard.tgz                                                 #SoC card brush package
│   ├── SOPHON_BSP_Technical_Reference_Manual.pdf
│   ├── SOPHON BSP开发参考手册.pdf
│   ├── sophon-img.MD5
│   ├── sophon-soc-libsophon_<x.y.z>_arm64.deb                     #Libsophon installation files corresponding to SoC platform
│   ├── sophon-soc-libsophon-dev_<x.y.z>_arm64.deb                 #Libsophon installation files corresponding to SoC platform
│   ├── system.tgz
│   └── tftp.tgz
├── sophon-mw_<date>_<hash>
│   ├── Multimedia FAQ.pdf
│   ├── Multimedia Technical Reference Manual.pdf
│   ├── Multimedia User Guide.pdf
│   ├── MULTIMEDIA使用手册.pdf
│   ├── MULTIMEDIA常见问题手册.pdf
│   ├── MULTIMEDIA开发参考手册.pdf
│   ├── release_version.txt
│   ├── sophon-mw_<x.y.z>_aarch64.tar.gz                          #Arm64 machine, sophon-mw installar for other LINUX systems
│   ├── sophon-mw_<x.y.z>_x86_64.tar.gz                           #Arm64 machine, sophon-mw installar for other LINUX systems
│   ├── sophon-mw.MD5
│   ├── sophon-mw-soc_<x.y.z>_aarch64.tar.gz
│   ├── sophon-mw-soc-sophon-ffmpeg_<x.y.z>_arm64.deb             #SoC platform, ffmpeg runtime environment installation files
│   ├── sophon-mw-soc-sophon-ffmpeg-dev_<x.y.z>_arm64.deb         #SoC platform, ffmpeg development environment installation files
│   ├── sophon-mw-soc-sophon-opencv_<x.y.z>_arm64.deb             #SoC platform, opencv runtime environment installation files
│   ├── sophon-mw-soc-sophon-opencv-dev_<x.y.z>_arm64.deb         #SoC platform, opencv development environment installation files
│   ├── sophon-mw-soc-sophon-sample_<x.y.z>_arm64.deb             #SoC platform, multimedia program sample file
│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_amd64.deb                 #X86x64 machine, installation files for ffmpeg runtime environment corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-ffmpeg_<x.y.z>_arm64.deb                 #arm64 machine, installation files for ffmpeg runtime environment corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_amd64.deb             #x86_64 machine, ffmpeg development environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-ffmpeg-dev_<x.y.z>_arm64.deb             #arm64 machine, ffmpeg development environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-opencv_<x.y.z>_amd64.deb                 #X86x64 machine, installation files for opencv runtime environment corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-opencv_<x.y.z>_arm64.deb                 #arm64 machine, installation files for opencv runtime environment corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_amd64.deb            #X86x64 machine, installation files for opencv runtime environment corresponding to CenterOS system
│   ├── sophon-mw-sophon-opencv-abi0_<x.y.z>_arm64.deb            #arm64 machine, opencv runtime environment installation file corresponding to CenterOS system
│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_amd64.deb        #x86x64 machine, opencv development environment installation file corresponding to CenterOS system
│   ├── sophon-mw-sophon-opencv-abi0-dev_<x.y.z>_arm64.deb        #arm64 machine, opencv development environment installation file corresponding to CenterOS system
│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_amd64.deb             #x86x64 machine, opencv development environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-opencv-dev_<x.y.z>_arm64.deb             #arm64 machine, opencv development environment installation file corresponding to Debian/Ubuntu system
│   ├── sophon-mw-sophon-sample_<x.y.z>_amd64.deb                 #x86x64 machine, sample files for multimedia programs
│   └── sophon-mw-sophon-sample_<x.y.z>_arm64.deb                 #arm64 machine, sample files of multimedia programs
├── sophon-pipeline_<date>_<hash>
│   ├── release_version.txt
│   ├── sophon-pipeline.MD5
│   └── sophon-pipeline_v<x.y.z>_586366b_20221027.tar.gz          #High performance inference framework based on pipeline
├── sophon-rpc_<date>_<hash>
│   ├── release_version.txt
│   ├── sophon-rpc_<x.y.z>_amd64.deb                              #x86x64 machine, sophon-rpc installation file corresponding to Debian/Ubuntu system
│   ├── sophon-rpc_<x.y.z>_arm64.deb                              #arm64 machine, sophon-rpc installation file corresponding to Debian/Ubuntu system
│   ├── sophon-rpc_<x.y.z>.tar.gz                                 #Sophon-rpc installation packages corresponding to other systems
│   ├── sophon-rpc.MD5
│   └── sophon-rpc使用指南.pdf
├── sophon-sail_<date>_<hash>
│   ├── release_version.txt
│   ├── Sophon_Inference_zh.pdf
│   ├── sophon-sail_<x.y.z>.tar.gz                                #Advanced interface library encapsulating BMLib, BMDecoder, BMCV, BMRuntime
│   └── sophon-sail.MD5
├── Sophon SDK文件清单.docx
├── tpu-kernel_<date>_<hash>
│   ├── release_version.txt
│   ├── tpu-kernel-1684x_v<x.y.z>-0d0e513e-221027.tar.gz          #Custom operator development tool
│   └── tpu-kernel.MD5
├── tpu-mlir_<date>_<hash>
│   ├── release_version.txt
│   ├── tpu-mlir.MD5
│   └── tpu-mlir_v<x.y.z>-<hash>-<date>.tar.gz                    #Compiler tool chain
├── tpu-nntc_<date>_<hash>
│   ├── release_version.txt
│   ├── tpu-nntc.MD5
│   └── tpu-nntc_v<x.y.z>-<hash>-<date>.tar.gz                    #Model compilation quantization tool chain
└── tpu-perf_v<x.y.z>
    ├── md5sum.txt
    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_aarch64.whl
    ├── tpu_perf-<x.y.z>-py3-none-manylinux2014_x86_64.whl
    └── tpu-perf-<x.y.z>.tar.gz                                   #Tools for performance analysis and accuracy verification of the model

```

#### SDK Main Modules [step]
* **Hardware Driver and Runtime Library LIBSOPHON** : contains libraries such as BMCV, BMRuntime, BMLib, etc., which are used to drive hardware such as VPP, TPU, etc. toperform operations such as image processing, tensor operations, model inference, etc
* **Multimedia library SOPHON-MW** : supports BM-OpenCV and BM-FFmpeg for hardware accelerate of Sophon devices, parsing of RTSP streams, GB28181 streams, video and image encoding and decoding
* **Model Compilation Quantization Tool Chain TPU-NNTC** : supports model conversion of Caffe,Tensorflow,Pytorch,MXNet,Darknet, Paddle Paddle, ONNX and other framework models; supports model quantization: original model -> FP32 UModel -> INT8 UModel -> INT8 BModel, and also provides auto-cali automatic quantization tool.
* **Tensor Operation and Image Processing Library BMCV** : colour space conversion, scale transformation, affine transformation, projection transformation, linear transformation, frame, JPEG coding, BASE64 coding, NMS, sorting, feature matching
* **Equipment Management BMLib** : basic interfaces: management of device Handle, memory management, data handling, API sending and synchronisation, A53 enable, etc
* **SAIL, the Arithmetic AI acceleration library** : supports high-level interfaces to Python/C++ and is wrapper for underlying library interfaces such as BMRuntime,BMCV,BMDecoder
* **Custom Operator Advanced Programming Library BMLang**: A C++ based high-level programming library for Sophon TPUs, decoupled from hardware information, no knowledge of hardware architecture is required, code is written using tensor data (bmlang::Tensor) and computation operations (bm- lang::Operator), and finally bmlang::compile or bmlang::compile_with_check is used to generate a BModel that the TPU can run; it also supports the use of the arm cpu in BM168X to implement arithmetic that is not yet supported by the TPU.
* **Algorithm Parallel Accelerated Programming Library TPUKernel**: underlying programming interface based on the Sophon chip’s underlying atomic operations interface, requiring familiarity with hardware architecture and instruction set
* **Model Performance and Accuracy Verification Tool TPUPerf**: enables performance analysis and accuracy validation of models

#### Details [step]
* [Get SDK and Detailed Documentation](https://developer.sophgo.com/site/index/material/all/all.html)

## Memory Management [step]

A portion of the memory of the AIBOX-1684X is allocated to NPU, VPP, and VPU devices. Therefore, the memory data obtained using commands such as `free -h` is inconsistent with the actual memory of the AIBOX-1684X.

If you need to view the detailed memory allocation settings, or if the default memory allocation conflicts with actual requirements and needs to be adjusted, you can refer to the following steps.

### 1. Install the Memory Allocation Tool

Obtain the memory allocation tool package from [Download Center](https://community.t-firefly.com/en/doc/download/248) and transfer it to the AIBOX-1684X. Run the following command to decompress the package:

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

### 2. View Memory Allocation Settings

After decompressing the package, run the following command to view the memory allocation settings:

```bash
cd memory_edit
./memory_edit.sh -p
```

The following is an example of the memory allocation settings. The result displayed under `Info: get max memory size ...` is the maximum memory that can be configured for each device, and the result displayed under `Info: get now memory size ...` is the memory currently allocated for each device.

```bash
linaro@aibox-1684x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1684x_se7_v1_mini.dts
Info: =======================================================================
Info: get ddr information ...
Info: ddr12_size 8589934592 Byte [8192 MiB]
Info: ddr3_size 4294967296 Byte [4096 MiB]
Info: ddr4_size 4294967296 Byte [4096 MiB]
Info: ddr_size 16384 MiB
Info: =======================================================================
Info: get max memory size ...
Info: max npu size: 0x1dbf00000 [7615 MiB]
Info: max vpu size: 0xc0000000 [3072 MiB]
Info: max vpp size: 0x100000000 [4096 MiB]
Info: =======================================================================
Info: get now memory size ...
Info: now npu size: 0xf6e00000 [3950 MiB]
Info: now vpu size: 0x80000000 [2048 MiB]
Info: now vpp size: 0xc0000000 [3072 MiB]
```

### 3. Modify Memory Allocation Settings

You can set the memory allocation by referring to the following commands. The three input parameters are decimal numbers of the sizes to be configured for NPU, VPU, and VPP in MiB; or hexadecimal values in Byte.

```bash
# decimal, in MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# hexadecimal, in bytes
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

After executing one of the above commands, check whether there is an Error in the output, and whether the sizes of the three parts in the output similar to the following are the same as the sizes to be configured.

```bash
Info: output configuration results ...
Info: vpu mem area(ddr3): 0x8000000 [128 MiB] 0x20000000 -> 0x27ffffff
Info: ion npu mem area(ddr1): 0x80000000 [2048 MiB] 0x24100000 -> 0xa40fffff
Info: ion vpu mem area(ddr3): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: ion vpp mem area(ddr4): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: =======================================================================
Info: start check memory size ...
Info: check npu size: 0x80000000 [2048 MiB]
Info: check vpu size: 0x80000000 [2048 MiB]
Info: check vpp size: 0x80000000 [2048 MiB]
Info: check edit size ok
Info: en_emmcfile ok
```

If the check is correct, save the current work, replace the boot image in the boot partition with the modified `emmcboot.itb` file by referring to the following operations, and finally restart the machine for the changes to take effect.

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

## Remote Desktop [step]

x11vnc is a tool that allows administrators to connect to the server's real X desktop directly through the VNC Viewer.


### Ubuntu
To use the X Desktop environment on Ubuntu system, you need to manually install it. Follow these steps, and during the installation process, select `lightdm` as the Display Manager:
```
sudo apt update
sudo apt install -y xserver-xorg-video-dummy x11vnc xfce4
sudo vim /etc/X11/xorg.conf
sudo reboot
```
Edit the Xorg configuration file:
```
Section "Device"
    Identifier  "Configured Video Device"
    Driver      "dummy"
    VideoRam 256000
EndSection

Section "Monitor"
    Identifier  "Configured Monitor"
    HorizSync 5.0 - 1000.0
    VertRefresh 5.0 - 200.0
    ModeLine "1920x1080" 148.50 1920 2448 2492 2640 1080 1084 1089 1125 +Hsync +Vsync
#    Modeline "1280x800" 24.15 1280 1312 1400 1432 800 819 822 841
EndSection

Section "Screen"
    Identifier  "Default Screen"
    Monitor     "Configured Monitor"
    Device      "Configured Video Device"
    DefaultDepth 24
    SubSection "Display"
    Depth 24
    Modes "1920x1080"
#    Modes "1280x800"
    EndSubSection
EndSection
```
Start the x11vnc service:

```shell
sudo x11vnc -display :0 -auth /var/lib/lightdm/.Xauthority &
```

You can now use a VNC client on your PC to remotely connect using the `$bm1684_ip:0` address.

To connect using VNC Viewer:

(1) Download and install [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) on your PC, based on your system.

(2) Open VNC Viewer and skip the login step.

<center>

<img alt="" src="../../../../bm1684_img/AIBOX-1684X/vnc-01.png" width="700">
</center>

(3) Enter the `$bm1684_ip:0` address of your AIBOX-1684X:

<center>

<img alt="" src="../../../../bm1684_img/AIBOX-1684X/ubuntu-vnc-02.png" width="800">
</center>

(4) Click on the circular button next to the `linaro` username, select `xfce`, and enter the username and password (both are `linaro`):

<center>

<img alt="" src="../../../../bm1684_img/AIBOX-1684X/ubuntu-vnc-03.png" width="800">
</center>

(5) You will successfully enter the xfce4 desktop environment.

<center>

<img alt="" src="../../../../bm1684_img/AIBOX-1684X/ubuntu-vnc-04.png" width="800">
</center>

## Qt Development [step]

### How to Develop Qt Programs and Display Them via HDMI

The overall process is divided into two steps:  

1. Cross-compile Qt programs
2. Set up the Qt runtime environment and run the program

#### 1. Cross-compile

Taking a cross-compilation host with the system as `Ubuntu18.04` as an example, the steps are as follows:

##### 1.1 Obtain Resources

Clone this [git repository](https://github.com/sophon-ai-algo/sophon-qt) using git clone.

Create and name a file called `se-build-server.yml` with the following content:

```
version: "3.3"
services:
    se-build-server:
        image: huangzhenming/se-build-server:V1.0
        container_name: sophon_build 
        privileged: true 
        network_mode: host 
        restart: always 
        volumes:
            - /data/your_working_dir:/workspace 
            - /data/toolchains/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu:/toolchains 
            - /etc/timezone:/etc/timezone:ro 
            - /etc/localtime:/etc/localtime:ro 
        command: ["/bin/ping", "localhost"] 
        user: linaro 
        working_dir: /workspace 
        environment: 
        - "PATH=$PATH:/toolchains/bin"
```    

 Download the necessary cross-compilation tools: [gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu](https://drive.google.com/drive/u/1/folders/1cC6hf_yKY8LsHSy1I8xbsR80fdVPSBaW)

Installation of the `docker` and `docker compose` tools varies from platform to platform, so please search for the relevant publicly available information on the Internet to install them, and we will not describe them here

Run docker (the runtime will automatically download the image from docker Hub, and the time required depends on the network speed, ensure a stable internet connection):
            
```
docker-compose -f se-build-server.yml  up -d
```  

After running, it will automatically create `/data/your_working_dir` and `/data/toolchains/` folders on your system, you can also modify the `yml` file to create folders in other paths.


Copy and extract the `git` package and the cross-compilation toolchain to the specified path:    
```
cp -r path/to/sophon-qt/qt-lib/qtbase-5.14.tgz /data/your_working_dir
cp -r path/to/sophon-qt/qt-example/SophUI/* /data/your_working_dir
cd /data/your_working_dir/ && tar xvf qtbase-5.14.tgz
cp -r path/to/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z /data/toolchains/
cd /data/toolchains/ && 7z x gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z
```  

* Cross-compile (using the `SophUI` source code as an example):    


    * enter docker:     


    ```
    docker ps -a  # Get the docker image and container information. The image name should have `huangzhenming` in it.
    docker exec -it  container_id  /bin/bash # enter docker 
    ```  


    * Perform the compilation(In docker):


    ```
    cd  /workspace
    ./install/bin/qmake SophUI.pro -spec linux-aarch64-gnu-g++ CONFIG+=qtquickcompiler
    make
    ```  


After compilation, the `SophUI` executable is generated, which can be executed directly on the device.            

#### 2. Run the Qt program

By following the above steps, you can get the executable file `SophUI` , which can be copied to the 1684 device by means of `scp`.

Before running, you need to write a script to set the environment variables as follows:  

```
#!/bin/sh
export QTDIR=/usr/lib/aarch64-linux-gnu 
export QT_QPA_FONTDIR=$QTDIR/fonts 
export QT_QPA_PLATFORM_PLUGIN_PATH=$QTDIR/qt5/plugins/
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
export QT_QPA_PLATFORM=linuxfb:fb=/dev/fl2000-0 
export QWS_MOUSE_PROTO=/dev/input/event3
./SophUI 
```  
After that, give the script permission to run, and execute it with the HDMI monitor plugged in.

**Note that if the 1684 is running on a `Debian` system, you will need to import the Qt libraries and other files separately, otherwise you will get an error, which is not the case with Ubuntu systems**.

Import it from the PC via `scp` and execute it on the 1684 device:  

```
scp PC@IP:/path/to/qtbase-5.14/lib/* /usr/lib/aarch64-linux-gnu/ 
mkdir -p /usr/lib/aarch64-linux-gnu/qt5
scp  PC@IP:/path/to/qtbase-5.14/plugins/ /usr/lib/aarch64-linux-gnu/qt5/
```
If the Qt program also contains fonts, you also need to [download](https://drive.google.com/drive/u/1/folders/1cC6hf_yKY8LsHSy1I8xbsR80fdVPSBaW) the font files here, and then copy them to the `/usr/lib/aarch64-linux-gnu/font` directory on the 1684 device. For example: `/usr/lib/aarch64-linux-gnu/fonts/wqy-microhei.ttc`.

