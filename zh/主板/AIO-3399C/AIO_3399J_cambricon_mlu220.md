

## 寒武纪MLU220-M.2边缘端智能加速卡支持



### 简介

MLU220-M.2 加速卡采用思元220芯片，芯片基于寒武纪MLUv02架构。加速卡采用标准M.2接口，理论峰值性能为8TOPS，功耗仅为8.25W。可以轻松实现终端设备和边缘段设备的AI赋能方案。

MLU220-M.2加速卡用于离线模型的部署，离线模型在MLU220上运行， 不依赖AI框架，只需要利用CNRT库，即可实现高效的AI推理运算。

如果需要对模型转化、量化、开发、精度调优等需要在配合MLU270在x86 PC上完成，然后生成离线模型，部署到MLU220加速卡上。

### 核心特性

#### 寒武纪MLUv02架构

- 多达16个张量核心，借助NOC片上网络提升并行效率
- 硬件片内数据压缩，提升缓存有效容量和带宽
- 更全面AI精度支持： INT16, INT8, INT4, FP32, FP16

#### 小个头大智能

- U盘大小，可以提供8路高清视频的实时智能分析

#### 计算弹性和可编程

- 高效支持多种神经网络
- 寒武纪Neuware软件栈和BANG语言编程环境，满足全面AI定制化要求
- 全面支持模型拆分并行，多模型并行，最大化计算效率



### MLU220 与 MLU270

#### MLU220产品定位

- 开发调试： 基于MLU270板卡，在线模式（使用AI框架），通用性高，可生成离线模型
- 业务部署： 基于MLU220板卡，离线模式（脱离AI框架），直接加载离线模型，性能更优

#### MLU220 离线部署

![](../../../rk3399_img/MLU220_Offline_Deploy.png)

#### MLU220 应用流程

![](../../../rk3399_img/MLU220_Process.png)

### MLU220 + Firefly AIO-3399J硬件平台

#### 系统固件更新

Firefly AIO-3399J操作系统使用Firefly定制的Ubuntu18.04.  系统中已经适配了MLU220 离线运行环境。包含以下几个部分：

- MLU220-M.2 内核驱动
- CNToolKit 运行库
- 编译工具链
- CNStream SDK / Demo。



固件下载地址如下：

[AIO-3399J Ubuntu18.04 MLU220固件](https://www.t-firefly.com/doc/download/31.html#other_174)

选择AIO-3399J-UBUNTU18.04-MLU220-GPT-**.img.7z

固件升级方法参照：

[AIO-3399J 固件升级](https://wiki.t-firefly.com/zh_CN/AIO-3399C/03-upgrade_firmware.html)



#### 验证运行环境

在系统启动完成进入桌面后，可以使用以下命令验证为了验证当前环境是否已经正确配置，如有异常，请与Firefly客服联系。

```shell
firefly@firefly:~$ lspci -v -d:0220
0002:21:00.0 Processing accelerators: Device cabc:0220
        Subsystem: Device cabc:0033
        Flags: bus master, fast devsel, latency 0, IRQ 114
        Memory at 381000000 (64-bit, prefetchable) [size=16M]
        Memory at 382000000 (64-bit, prefetchable) [size=8M]
        Memory at 380c00000 (64-bit, prefetchable) [size=2M]
        Capabilities: <access denied>
        Kernel driver in use: cambricon-pci-drv
        Kernel modules: cambricon_drv

firefly@firefly:~$ ls /dev/cambricon_*
/dev/cambricon_ctl  /dev/cambricon_dev0

firefly@firefly:~$ cnmon
Wed May 19 10:33:51 2021
+-----------------------------------------------------------------------------+
| CNMON 1.16.0                                                                |
+------------------------------+----------------------+-----------------------+
| Card  VF  Name      Firmware | Inited        Driver | Util        Ecc-Error |
| Fan   Temp     Pwr:Usage/Cap |         Memory-Usage |         vMemory-Usage |
|==============================+======================+=======================|
| 0     /   MLU220-M2   v1.1.1 | On            v4.6.9 | 0%          N/A       |
|  0%   51C     3.70 W/ 8.25 W |   967 MiB/  4096 MiB | 10240 MiB/1048576 MiB |
+------------------------------+----------------------+-----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  Card  VF  PID    Command Line                           MLU Memory Usage   |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```



#### 运行CNStream Demo (yolov3)

```shell
firefly@firefly:~$ cd /home/firefly/cnstream/samples/demo/detection/mlu220
firefly@firefly:~/cnstream/samples/demo/detection/mlu220$ ./run_yolov3_mlu220.sh
yolov3_4c4b_argb_220_v1.5.0.cambricon exists.
/home/firefly/cnstream/samples/demo/detection/mlu220
/home/firefly/cnstream/samples/demo/detection/mlu220
Rga built version:06fc7c4
CNRT: 4.7.5 3d3546b
WARNING: Logging before InitCNStreamLogging() is written to STDERR
CNSTREAM REFLEX_OBJECT I0519 10:36:26.904601  2121] Register object named [PreprocCpu]
CNSTREAM REFLEX_OBJECT I0519 10:36:26.906293  2121] Register object named [ObjPreprocCpu]
CNSTREAM REFLEX_OBJECT I0519 10:36:26.906965  2121] Register object named [PreprocYolov3]
CNSTREAM REFLEX_OBJECT I0519 10:36:26.907565  2121] Register object named [VideoPreprocCpu]
...

```



### CNToolKit 运行库介绍

CNToolKit运行库类组件主要为用户提供部署环境和开发环境中的运行时支持。在MLU220中，只支持边缘端的部署环境。

- CNRT

  - CNRT（Cambricon Neuware Runtime Library，寒武纪运行时库）提供了一套面向MLU（Machine Learning Unit，寒武纪机器学习单元）设备的高级别的接口，用于主机与MLU设备之间的交互。CNRT作为寒武纪软件系统最底层支撑，所有其他的寒武纪软件运行都需要调用CNRT接口。
  - 有关CNRT的详细开发文档，请参见[寒武纪官方文档-CNRT](https://www.cambricon.com/docs/cnrt/user_guide_html/index.html)

- CNCodec

  - CNCodec（Cambricon Neuware Codec，寒武纪硬件编解码）是一套封装了视频编解码和图片编解码的SDK接口。
  - CNCodec典型应用场景如：带有CNCodec板卡的主机接收IP Camera传输过来的码流数据，CNCodec接收到码流数据后，将数据输入给VPU（Video Processor Unit，视频处理单元）或JPU（JPEG Processor Unit，JPEG处理单元）进行硬件编解码， 编解码得到的图片数据送给MLU（Machine Learning Unit，寒武纪机器学习单元）进行推理，推理完成后返回推理结果至主机。
  - 有关CNCodec的详细开发文档，请参见[寒武纪官方文档-CNCodec](https://www.cambricon.com/docs/cncodec/cncodec_function/cncodec_function.html)

- CNDev

  - CNDev（Cambricon Neuware Device Interface，寒武纪设备接口）是一套支持主机端应用程序获取寒武纪芯片硬件信息的软件接口。 通过CNDev接口，用户可以对设备管理实现定制化需求，比如云服务平台的设备管理、自动化测试平台的设备管理等。

    CNDev主要实现了以下功能：

    - 获取设备硬件信息，比如：设备名称、设备利用率、板卡功耗、板卡温度、板卡带宽、风扇转速等。
    - 评测应用程序在寒武纪硬件设备上的开销，比如：内存占用、MLU利用率等。
    - 提供获取设备间拓扑逻辑以及CPU亲和性等相关信息的功能。

  - 有关CNDev的详细开发文档，请参见[寒武纪官方文档-CNDev](https://www.cambricon.com/docs/cndev/index.html)

- CNDrv

  - CNDRV API（Cambricon Driver Application Programming Interface，寒武纪驱动应用程序接口）提供了一套面向MLU（Machine Learning Unit，机器学习单元）设备的接口，用于主机与MLU设备之间的交互。 CNDRV API作为寒武纪软件系统的底层支撑，向下封装了对更底层驱动的操作，向上提供了可以调用的接口，通过该接口实现了上层对底层驱动的操作。 上层框架既可以间接通过CNRT（Cambricon Neuware Runtime Library，寒武纪运行时库）调用CNDRV API进行软件编程，也可以直接调用CNDRV API进行软件编程。
  - 有关CNDrv的详细开发文档，请参见[寒武纪官方文档-CNDrv](https://www.cambricon.com/docs/cndrv/index.html)



### CNStream SDK

CNStream是面向寒武纪开发平台的数据流处理SDK。用户可以根据CNStream提供的接口，开发实现自己的组件。还可以通过组件之间的互连，灵活地实现自己的业务需求。CNStream能够大大简化寒武纪深度学习平台提供的推理和其他处理，如视频解码、神经网络图像前处理的集成。也能够在兼顾灵活性的同时，充分发挥寒武纪MLU（Machine Learning Unit 机器学习处理器）的硬件解码和机器学习算法的运算性能。

CNStream 开源项目地址：

https://github.com/cambricon/cnstream

CNStream 详细开发文档：

https://www.cambricon.com/docs/cnstream/developer_guide_html/index.html

https://www.cambricon.com/docs/cnstream/user_guide_html/overview/Overview.html



### 开发流程

MLU220-M.2加速卡用于离线模型的部署，如果需要对模型转化、量化、开发、精度调优等需要在配合MLU270在x86 PC上完成。

配合寒武纪人工智能开发平台（Cambricon NeuWare®），用户在x86 PC上，利用MLU270智能加速卡，完成的开发工作，保存生成的离线模型。

然后再将保存生成的离线模型，部署到Firefly AIO-3399J平台上。

更多关于Cambricon NeuWare的文档，请参阅[寒武纪官方文档](https://www.cambricon.com/index.php?m=content&amp;c=index&amp;a=lists&amp;catid=71)
