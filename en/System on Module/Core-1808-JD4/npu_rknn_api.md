# RKNN API

Rockchip provides a set of RKNN API SDK, which is a set of acceleration scheme for NPU hardware of neural network based on RK1808 Linux, and can provide general  acceleration support for AI-related applications developed with RKNN API.

For the introduction of RKNN API SDK related APIs, please refer to `Rockchip_RK1808_Developer_Guide_Linux_RKNN_EN.pdf` in the SDK directory `docs/Linux/NPU`. The following is the introduction of RKNN API configuration and usage. For details, please refer to the examples in RKNN API.

## 1. Linux

The application only needs to include the header file and the dynamic library to write the relevant AI application.

### Buildroot

The SDK provides MobileNet image classification, MobileNet SSD target detection, and Yolo v3 target detection demo for Linux platform. These demos can provide a reference for customers to develop their own AI applications based on the RKNN SDK. Demo code is located at `<rk1808-linux-sdk>/external/rknpu/rknn/rknn_api/examples/rknn_mobilenet_demo` as an example to explain how to get started quickly.

#### Demo use

1. Compile Demo

	```
    cd examples / rknn_mobilenet_demo
    mkdir build && cd build
    cmake ..
    make && make install
    cd –
	```
2. Deploy to RK1808 device

	```
    adb push install / rknn_mobilenet_demo / userdata /
	```

3. Run Demo
	```
    adb shell
    cd / userdata / rknn_mobilenet_demo
    ./rknn_mobilenet_demo mobilenet_v1.rknn dog_224x224.jpg
	```

#### Configuration

For details on how to configure rknn_api to compile the application, refer to the CMakeLists.txt file under the Demo.

The rknn_api header files and dynamic libraries are in the `<rk1808-linux-sdk>/external/rknpu/rknn/rknn_api/examples/libs/librknn_api` directory in the SDK.

```
├── include
│ └── rknn_api.h
└── lib64
  └── librknn_api.so
```

* Reference the dynamic library

```
LDFLAGS + = -lrknn_api
```

* Reference the rknn_api header file

```
#include "rknn_api.h"
```