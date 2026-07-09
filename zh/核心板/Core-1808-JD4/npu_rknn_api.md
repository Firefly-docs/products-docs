# RKNN API

Rockchip提供了一套RKNN API SDK，该SDK为基于 RK1808 Linux 的神经网络NPU硬件的一套加速方案，可为采用RKNN API 开发的AI相关应用提供通用加速支持。

**RKNN API SDK相关API介绍请参考文档在SDK目录`docs/Linux/NPU`下的《Rockchip_RK1808_Developer_Guide_Linux_RKNN_CN.pdf》,以下为RKNN API配置使用介绍,详细内容请参考RKNN API中的示例。**

## 1. Linux系统

应用程序只需要包含该头文件和动态库，就可以编写相关的AI应用。

### Buildroot

SDK 提供了 Linux 平台的 MobileNet 图像分类、MobileNet SSD 目标检测以及 Yolo v3 目标检测 Demo。这些 Demo 能够为客户基于 RKNN SDK 开发自己的 AI 应用提供参考。Demo代码位于 `<rk1808-linux-sdk>/external/rknpu/rknn/rknn_api/examples/rknn_mobilenet_demo` 为例来讲解如何快速上手运行。

#### Demo使用

1. 编译 Demo
	```
	cd examples/rknn_mobilenet_demo
	mkdir build && cd build
	cmake ..
	make && make install
	cd –
	```
2. 部署到 RK1808 设备
	```
	adb push install/rknn_mobilenet_demo /userdata/
	```
3. 运行 Demo
	```
	adb shell
	cd /userdata/rknn_mobilenet_demo
	./rknn_mobilenet_demo mobilenet_v1.rknn dog_224x224.jpg
	```
#### 配置

具体如何配置 rknn_api 编译应用程序，可以参考该 Demo 下的 `CMakeLists.txt` 文件。

rknn_api的头文件和动态库在SDK中的 `<rk1808-linux-sdk>/external/rknpu/rknn/rknn_api/examples/libs/librknn_api` 目录下。
```
├── include
│   └── rknn_api.h
└── lib64
    └── librknn_api.so
```

* 引用动态库
```
LDFLAGS += -lrknn_api
```
* 引用 rknn_api 头文件
```
#include "rknn_api.h"
```