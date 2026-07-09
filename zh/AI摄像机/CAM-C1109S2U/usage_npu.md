# NPU 使用

`RK1126/RK1109` NPU 进行模型推理时必须使用` RKNN `模型。如果用户使用自己研发的算法模型，需要熟悉 RKNN 
开发流程，把自己的模型转换成 RKNN 再进行使用。本章针对这类用户讲解一下 RKNN 开发配套工具的使用。

## RKNN 介绍

RKNN 是 Rockchip npu 平台使用的模型类型，以` .rknn `后缀结尾的模型文件。Rockchip 提供了完整了模型转换
 Python 工具，方便用户将自主研发的算法模型转换成 RKNN 模型，同时 Rockchip 也提供了` C/C++ `和` Python `
 API 接口。

## RKNN-Toolkit

### 工具介绍
`RKNN-Toolkit `是为用户提供在 PC、 Rockchip NPU 平台上进行模型转换、推理和性能评估的开发套件,用户通过该工具提供的 Python 接口可以便捷地完成以下功能:

* <font color=blue>**模型转换**</font>：支持` Caffe`、`TensorFlow`、`TensorFlow Lite`、`ONNX`、`Darknet`、`Pytorch`、`MXNet`
模型转成 RKNN 模型,支持 RKNN 模型导入导出,后续能够在 Rockchip NPU 平台上加
载使用。从` 1.2.0 `版本开始支持多输入模型。从` 1.3.0 `版本开始支持 Pytorch 和 MXNet。

* <font color=blue>**量化功能**</font>：支持将浮点模型转成量化模型, 目前支持的量化方法有`非对称量化`( asymmetric_quantized-u8 ) , 
`动态定点量化` ( dynamic_fixed_point-8 和 dynamic_fixed_point-16)。从` 1.0.0 `版本开始,
RKNN-Toolkit 开始支持`混合量化`功能。

* <font color=blue>**模型推理**</font>：能够在 PC 上模拟 Rockchip NPU 运行 RKNN 模型并获取推理结果;也可以将
RKNN 模型分发到指定的 NPU 设备上进行推理。

* <font color=blue>**性能评估**</font>：能够在 PC 上模拟 Rockchip NPU 运行 RKNN 模型,并评估模型性能(包括总
耗时和每一层的耗时);也可以将 RKNN 模型分发到指定 NPU 设备上运行,以评估模型
在实际设备上运行时的性能。

* <font color=blue>**内存评估**</font>：评估模型运行时对系统和 NPU 内存的消耗情况。使用该功能时,必须将 RKNN
模型分发到 NPU 设备中运行,并调用相关接口获取内存使用信息。从` 0.9.9 `版本开始支持
该功能。

* <font color=blue>**模型预编译**</font>：通过预编译技术生成的 RKNN 模型可以减少在硬件平台上的加载时间。对
于部分模型,还可以减少模型尺寸。但是预编译后的 RKNN 模型只能在 NPU 设备上运行。
目前只有 x86_64 Ubuntu 平台支持直接从原始模型生成预编译 RKNN 模型。 RKNN-Toolkit
从` 0.9.5 `版本开始支持模型预编译功能,并在` 1.0.0 `版本中对预编译方法进行了升级,升级
后的预编译模型无法与旧驱动兼容。从` 1.4.0 `版本开始,也可以通过 NPU 设备将普通 RKNN
模型转成预编译 RKNN 模型。

* <font color=blue>**模型分段**</font>：该功能用于多模型同时运行的场景下,可以将单个模型分成多段在 NPU 上执
行,借此来调节多个模型占用 NPU 的执行时间,避免因为一个模型占用太多执行时间,
1而使其他模型得不到及时执行。RKNN-Toolkit 从` 1.2.0 `版本开始支持该功能。该功能必须
在带有 Rockchip NPU 的硬件上使用,且 NPU 驱动版本要大于` 0.9.8`。

* <font color=blue>**自定义算子功能**</font>：如果模型含有 RKNN-Toolkit 不支持的算子(operator),那么在模型转
换阶段就会失败。这时候可以使用自定义算子功能来添加不支持的算子,从而使模型能正
常转换和运行。RKNN-Toolkit 从` 1.2.0 `版本开始支持该功能。自定义算子的使用和开发请
参考`《Rockchip_Developer_Guide_RKNN_Toolkit_Custom_OP_CN》`文档。

* <font color=blue>**量化精度分析功能**</font>：该功能将给出模型量化前后每一层推理结果的欧氏距离或余弦距离,
以分析量化误差是如何出现的,为提高量化模型的精度提供思路。该功能从` 1.3.0 `版本开
始支持。`1.4.0 `版本增加逐层量化精度分析子功能,将每一层运行时的输入指定为正确的
浮点值,以排除逐层误差积累,能够更准确的反映每一层自身受量化的影响。

* <font color=blue>**可视化功能**</font>：该功能以图形界面的形式呈现 RKNN-Toolkit 的各项功能,简化用户操作步
骤。用户可以通过填写表单、点击功能按钮的形式完成模型的转换和推理等功能,而不需
要再去手动编写脚本。有关可视化功能的具体使用方法请参考
`《Rockchip_User_Guide_RKNN_Toolkit_Visualization_CN》`文档。`1.3.0 `版本开始支持该功
能。`1.4.0 `版本完善了对多输入模型的支持,并且支持 RK1806, RV1109, RV1126 等新的
Rockchip NPU 设备。

* <font color=blue>**模型优化等级功能**</font>：RKNN-Toolkit 在模型转换过程中会对模型进行优化,默认的优化选项
可能会对模型精度产生一些影响。通过设置优化等级,可以关闭部分或全部优化选项。有
关优化等级的具体使用方法请参考 config 接口中` optimization_level `参数的说明。该功能从
`1.3.0 `版本开始支持。

### 环境依赖

* <font color=blue>**系统支持**</font>：Ubuntu 16.04 x64（以上）、Window 7 x64（以上）、Mac OS X 10.13.5 x64 （以上）、
Debian 9.8 （x64）以上

* <font color=blue>**Python 版本**</font>：3.5/3.6/3.7

* <font color=blue>**Python 依赖**</font>：
```
'numpy == 1.16.3'
'scipy == 1.3.0'
'Pillow == 5.3.0'
'h5py == 2.8.0'
'lmdb == 0.93'
'networkx == 1.11'
'flatbuffers == 1.10',
'protobuf == 3.6.1'
'onnx == 1.4.1'
'onnx-tf == 1.2.1'
'flask == 1.0.2'
'tensorflow == 1.11.0' or 'tensorflow-gpu'
'dill == 0.2.8.2'
'ruamel.yaml == 0.15.81'
'psutils == 5.6.2'
'ply == 3.11'
'requests == 3.11'
'pytorch == 1.2.0'
'mxnet == 1.5.0'
```

<font color=red>**PS**</font>：
1. Windows 只提供` Python3.6 `的安装包。
2. MacOS 提供` Python3.6 `和` Python3.7 `的安装包。
3. ARM64 平台(安装 Debian 9 或 10 操作系统)提供` Python3.5 `(Debain 9)和` Python3.7`
(Debian10)的安装包。
4. 除 MacOS 平台外,其他平台的 scipy 依赖为>=1.1.0。

### 快速上手

#### PC 主机

测试环境使用` Ubuntu 16.04 x86_64 PC `主机。其他平台可以参考 sdk/external/rknn-toolkit/doc/Rockchip_Quick_Start_RKNN_Toolkit_V1.4.0_XX.pdf。

* RKNN-Toolkit 安装
```
# 安装 python 3.5
sudo apt-get install python3.5
# 安装 pip3
sudo apt-get install python3-pip
# 获取 RKNN-Toolkit 安装包,然后执行以下步骤
cd sdk/external/rknn-toolkit/
cp sdk/external/rknn-toolkit ./ -rf
cd rknn-toolkit/package/
pip3 install tensorflow==1.11.0
pip3 install mxnet==1.5.0
pip3 install torch==1.2.0 torchvision==0.4.0
pip3 install opencv-python
pip3 install gluoncv
# 安装 RKNN-Toolkit
sudo pip3 install rknn_toolkit-1.4.0-cp35-cp35m-linux_x86_64.whl
# 检查是否安装成功，import rknn 库
rk@rk:~/rknn-toolkit-v1.4.0/package$ python3
>>> from rknn.api import RKNN
>>>
```
* 设备默认没有开启 OTG 功能，如需要请先在内核开启 OTG，编译升级内核。
```
# sdk/kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi
&usbdrd_dwc3 {
	status = "okay";
	dr_mode = "otg";  # 开启 OTG
	extcon = <&u2phy0>;
};
```

* USB OTG 连接主机和设备运行 demo
```
cd examples/tflite/mobilenet_v1/
daijh@daijh:~/p/sdk/external/rknn-toolkit/examples/tflite/mobilenet_v1$ python3.6 ./test.py 
--> config model
done
--> Loading model
done
--> Building model
W The channel_mean_value filed will not be used in the future!
done
--> Export RKNN model
done
--> Init runtime environment
I NPUTransfer: Starting NPU Transfer Client, Transfer version 2.0.0 (8f9ebbc@2020-04-03T09:12:30)
D RKNNAPI: ==============================================
D RKNNAPI: RKNN VERSION:
D RKNNAPI:   API: 1.4.0 (b4a8096 build: 2020-08-12 10:15:19)
D RKNNAPI:   DRV: 1.5.2 (e67e5cb build: 2020-12-03 15:04:52)
D RKNNAPI: ==============================================
done
--> Running model
mobilenet_v1
-----TOP 5-----
[156]: 0.8603515625
[155]: 0.0833740234375
[205]: 0.0123443603515625
[284]: 0.00726318359375
[260]: 0.002262115478515625

done
--> Begin evaluate model performance
W When performing performance evaluation, inputs can be set to None to use fake inputs.
========================================================================
                               Performance                              
========================================================================
Total Time(us): 5573
FPS: 179.44
========================================================================

done
daijh@daijh:~/p/sdk/external/rknn-toolkit/examples/tflite/mobilenet_v1$
```

* 除了` python `接口外，我们也提供了模型推理的` C/C++ `接口，用户可以在 PC 上完成模型转换再到板子上
使用` C/C++ `完成模型推理。以下是 demo 运行。
```
# 编译前需要修改好交叉编译器的路径,vim build.sh 修改 GCC_COMPILER
# GCC_COMPILER=/home/daijh/p/sdk/prebuilts/gcc/linux-x86/arm/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf 
# 这里是我本地的 32 位交叉编译工具的路径，用户需要自己修改成 SDK 中交叉编译工具的路径。
daijh@daijh:~$ cd sdk/external/rknpu/rknn/rknn_api/examples/rknn_mobilenet_demo
daijh@daijh:sdk/external/rknpu/rknn/rknn_api/examples/rknn_mobilenet_demo$ ./build.sh

# 把编译好的 demo 放到设备中
adb push rknn_mobilenet_demo/ /

# 运行 demo
cd rknn_mobilenet_demo
[root@RV1126_RV1109:/rknn_mobilenet_demo]# ./build/rknn_mobilenet_demo ./model/mobilenet_v1_rv1109_rv1126.rknn ./model/dog_224x224.jpg
model input num: 1, output num: 1
input tensors:
index=0 name= n_dims=4 dims=[1 224 224 3] n_elems=150528 size=150528 fmt=0 type=3 qnt_type=2 fl=127 zp=127 scale=0.007843
output tensors:
index=0 name= n_dims=2 dims=[0 0 1 1001] n_elems=1001 size=2002 fmt=0 type=1 qnt_type=0 fl=127 zp=127 scale=0.007843
rknn_run
155 - 0.091736
156 - 0.851074
205 - 0.013588
```
#### RV1126 主机

测试环境使用` RV1126 `主机。文件系统是 ` Debian10 `。以下操作均在 ` RV1126 `上执行。

* Firefly Debian10 固件 RKNN Toolkit Lite 安装步骤：<br>

一、 安装依赖 ` numpy / psutils / ruamel.yaml `
```
# 如果没有装 pip3 ，请先用 sudo apt-get update && sudo apt-get install python3-pip 装下
pip3 install numpy==1.16.3
pip3 install psutil==5.6.2
pip3 install ruamel.yaml==0.15.81
```

二、 安装 ` opencv-python ` 。用 pip3 安装会一直失败，所以直接在网上下载包。

点击：[资源下载](https://www.t-firefly.com/doc/download/98.html)下载 rknn-toolkit-lite.rar 并拷贝到 rv1126 系统上解压。

```
# 先装以下依赖，两个 deb 包已经放在 rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires/ 目录下
sudo apt-get install multiarch-support unrar
unrar x rknn-toolkit-lite.rar
cd rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/requires/

sudo dpkg -i libjasper1_1.900.1-debian1-2.4+deb8u6_armhf.deb
sudo dpkg -i libjasper-dev_1.900.1-debian1-2.4+deb8u6_armhf.deb
sudo apt-get install libhdf5-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libqtgui4
sudo apt-get install libqt4-test
pip3 install opencv_python-4.0.1.24-cp37-cp37m-linux_armv7l.whl
```

三、安装 RKNN Toolkit Lite
```
# 使用以下命令安装 RKNN Toolkit Lite
cd rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/packages

pip3 install rknn_toolkit_lite-1.7.0.dev_0cfb22-cp37-cp37m-linux_armv7l.whl
```

四、 跑 example（需要切换到 root 用户执行）
```
cd rknn-toolkit-lite/rknn-toolkit-lite-v1.7.0.dev_0cfb22/examples-lite/inference_with_lite
python3 test.py

# 结果输出如下：
root@firefly:/home/firefly/rknn-toolkit-lite-v1.7.0.dev_0cfb22/examples-lite/inference_with_lite# python3 test.py
--> list devices:
*************************
None devices connected.
*************************
done
--> query support target platform
**************************************************
Target platforms filled in RKNN model:         ['RV1109']
Target platforms supported by this RKNN model: ['RK1109', 'RK1126', 'RV1109', 'RV1126']
**************************************************
done
--> Load RKNN model
done
--> Init runtime environment
done
--> get sdk version:
==============================================
RKNN VERSION:
  API: librknn_runtime version 1.6.0 (6523e57 build: 2021-01-15 15:56:31 base: 1126)
  DRV: 6.4.3.5.293908
==============================================

done
--> Running model
resnet18
-----TOP 5-----
[812]: 0.9993900656700134
[404]: 0.0004593880439642817
[657 833]: 2.9284517950145528e-05
[657 833]: 2.9284517950145528e-05
[895]: 1.850890475907363e-05

done
```

### 开发文档

当你安装好` RKNN-Toolkit `并且通过 demo 初步了解和验证了开发流程后，你可以查看详细的 RKNN 开发 API 去完成自己的开发了。

* <font color=blue>**RKNN-Toolkit 文档**</font>：sdk/external/rknn-toolkit/doc
* <font color=blue>**C/C++ API 文档**</font>：sdk/external/rknpu/rknn/doc
