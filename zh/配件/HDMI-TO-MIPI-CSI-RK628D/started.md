# HDMI-TO-MIPI-CSI-RK628D 驱动板

## 一、产品介绍
### 产品简介
#### HDMI TO MIPI CSI 驱动板
HDMI TO MIPI CSI驱动板，采用视频桥接芯片RK628D，可实现HDMI视频信号转换为MIPI CSI信号的转换需求，可极大简化硬件设计，提高硬件设计效率，并节约整机成本。可应用于智能微型投影仪、智能显示屏、视频采集转换类产品。


![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_front.jpg)

### 发货清单(仅供参考)

![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_shipping_list.png)


### 规格参数
* 尺寸大小：43.5mm * 30 mm
* HDMI RX 接口：
  * 兼容HDMI 1.4/HDMI 2.0
  * 支持8/10bit 视频格式
  * 支持RGB888/YUV420
  * 最大支持分辨率4K@60fps (YUV420)

* MIPI CSI 接口：
  * 兼容MIPI DPHY V1.2
  * 支持YUV422
  * 主要分辨率支持：3840x2160@30fps，1920x1080@60fps，1280x720@60fps，720x480@60fps

* LINEOUT 接口：
  * 同一路音频输出：3.5mm 耳机接口 * 1、3P-2mm * 1、2P-1.25mm * 1

### 接口定义
![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_interface.jpg)

Line Out：将声卡处理后的模拟信号通过此接口输出到音频设备上。
<!--
### 接口定义
### 产品选型
-->

## 二、使用方法

<!--
### 使用说明
-->
<font color='red'>主板支持列表详见[固件制作](#gu-jian-zhi-zuo)章节</font>

### 硬件连接

#### RK3566/RK3568/RK3588/RK3588S/RK3576 系列主板
![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_connection.jpg)

#### RK3399系列主板
![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_connection_30p-to-24p.jpg)


<font color='red'>注意:<br />1. 请在驱动板和开发板处于断电状态操作此步骤，避免烧坏驱动板和开发板;<br />2. RK3399 系列主板如存在两个MIPI CSI接口，默认接MIPI CSI0。
</font>

### Android 使用HDMI-IN
系统默认内置HdmiIn2的应用，如图：
![](../../../modules_img/HDMI-TO-MIPI-CSI-RK628D/rk628d_HdmiIn2.png)

点击进入应用后，就会有HDMI-IN 的图像显示，音频会从驱动板上的LineOut输出，最大支持输出分辨率3840x2160@30fps。

### Linux 使用 HDMI-IN

在设备上运行如下脚本：

```bash
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/aarch64-linux-gnu/gstreamer-1.0
WIDTH=1920
HEIGHT=1080
SINK=xvimagesink

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT} ! videoconvert ! $SINK &

wait
```

## 三、固件与资料下载
相关文档和固件下载，见官网的[资料下载](https://www.t-firefly.com/doc/download/206.html)。

<!--
### 文档下载
### 配件图纸
### 软件工具
### 固件下载
-->

## 四、入门教程
### 固件烧写

| 主控 | USB 线刷 | SD 卡升级 |
| ---- | ---- | ---- |
|RK3566|[AIO-3566JD4](../../核心板/Core-3566JD4/03-upgrade_firmware.md), [ROC-RK3566-PC](../../主板/ROC-RK3566-PC/03-upgrade_firmware.md)| [AIO-3566JD4](../../核心板/Core-3566JD4/05-upgrade_firmware_sd.md), [ROC-RK3566-PC](../../主板/ROC-RK3566-PC/05-upgrade_firmware_sd.md) | 
|RK3568|[AIO-3568J](../../核心板/Core-3568J/03-upgrade_firmware.md), [ROC-RK3568-PC](../../主板/ROC-RK3568-PC/03-upgrade_firmware.md), [ROC-RK3568-PC SE](../../主板/ROC-RK3568-PC-SE/03-upgrade_firmware.md) | [AIO-3568J](../../核心板/Core-3568J/05-upgrade_firmware_sd.md), [ROC-RK3568-PC](../../主板/ROC-RK3568-PC/05-upgrade_firmware_sd.md), [ROC-RK3568-PC SE](../../主板/ROC-RK3568-PC-SE/05-upgrade_firmware_sd.md)| 
|RK3588|[ITX-3588J](../../核心板/Core-3588J/upgrade_firmware.md), [AIO-3588Q](../../核心板/iCore-3588Q/upgrade_firmware.md) | [ITX-3588J](../../核心板/Core-3588J/upgrade_firmware_sd.md), [AIO-3588Q](../../核心板/iCore-3588Q/upgrade_firmware_sd.md) |
|RK3588S| [AIO-3588SJD4](../../核心板/Core-3588SJD4/upgrade_firmware.md), [ROC-RK3588S-PC](../../主板/ROC-RK3588S-PC/upgrade_firmware.md),[AIO-3588SG](../../核心板/Core-3588SG/upgrade_firmware.md) | [AIO-3588SJD4](../../核心板/Core-3588SJD4/upgrade_firmware_sd.md), [ROC-RK3588S-PC](../../主板/ROC-RK3588S-PC/upgrade_firmware_sd.md), [AIO-3588SG](../../核心板/Core-3588SG/upgrade_firmware_sd.md)|
|RK3576|[ROC-RK3576-PC](../../主板/ROC-RK3576-PC/upgrade_firmware.md),[AIO-3576Q](../../核心板/iCore-3576Q/upgrade_firmware.md) | [ROC-RK3576-PC](../../主板/ROC-RK3576-PC/upgrade_firmware_sd.md),[AIO-3576Q](../../核心板/iCore-3576Q/upgrade_firmware_sd.md) |

### 固件制作

#### RK3566 系列

|  系统   |  板卡型号 | 
|  ----  | ----  | 
| Android11.0 | [AIO-3566JD4](../../核心板/Core-3566JD4/compile_android11.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi), [ROC-RK3566-PC](../../主板/ROC-RK3566-PC/compile_android11.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi) |
| Ubuntu20.04 | [AIO-3566JD4](../../核心板/Core-3566JD4/linux_compile.md#xuan-ze-bian-yi-pei-jian), [ROC-RK3566-PC](../../主板/ROC-RK3566-PC/linux_compile.md#xuan-ze-bian-yi-pei-jian) |

#### RK3568 系列

|  系统   |  板卡型号 | 
|  ----  | ----  | 
| Android11.0 | [AIO-3568J](../../核心板/Core-3568J/compile_android11.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi), [ROC-RK3568-PC](../../主板/ROC-RK3568-PC/compile_android11.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi), [ROC-RK3568-PC SE](../../主板/ROC-RK3568-PC-SE/compile_android11.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi) |
| Ubuntu20.04 | [AIO-3568J](../../核心板/Core-3568J/linux_compile.md#xuan-ze-bian-yi-pei-jian), [ROC-RK3568-PC](../../主板/ROC-RK3568-PC/linux_compile.md#xuan-ze-bian-yi-pei-jian), [ROC-RK3568-PC SE](../../主板/ROC-RK3568-PC-SE/linux_compile.md#xuan-ze-bian-yi-pei-jian) |

#### RK3588 系列

|  系统   |  板卡型号 |
|  ----  | ----  |
| Android12.0 | [ITX-3588J](../../核心板/Core-3588J/android_compile_android12.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi), [AIO-3588Q](../../核心板/iCore-3588Q/android_compile_android12.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi) |
| Android14.0 | [ITX-3588J](../../核心板/Core-3588J/android_compile_android14.0_firmware.md#core-3588j-chan-pin-bian-yi-fang-fa), [AIO-3588Q](../../核心板/iCore-3588Q/android_compile_android14.0_firmware.md#icore-3588q-chan-pin-bian-yi-fang-fa)|
| Linux-kernel6.1 | [ITX-3588J](../../核心板/Core-3588J/linux6.1_compile.md), [AIO-3588Q](../../核心板/iCore-3588Q/linux6.1_compile.md)|

#### RK3588S 系列

|  系统   |  板卡型号 |
|  ----  | ----  |
| Android12.0 | [AIO-3588SJD4](../../核心板/Core-3588SJD4/android_compile_android12.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi), [ROC-RK3588S-PC](../../主板/ROC-RK3588S-PC/android_compile_android12.0_firmware.md#hdmi-to-mipi-csi-rk628d-bian-yi) |
| Android14.0 | [AIO-3588SJD4](../../核心板/Core-3588SJD4/android_compile_android14.0_firmware.md#core-3588sjd4-chan-pin-bian-yi-fang-fa), [ROC-RK3588S-PC](../../主板/ROC-RK3588S-PC/android_compile_android14.0_firmware.md#roc-rk3588s-pc-chan-pin-bian-yi-fang-fa), [AIO-3588SG](../../核心板/Core-3588SG/android_compile_android14.0_firmware.md#core-3588sg-chan-pin-bian-yi-fang-fa) |
| Linux-kernel6.1 | [AIO-3588SJD4](../../核心板/Core-3588SJD4/linux6.1_compile.md), [ROC-RK3588S-PC](../../主板/ROC-RK3588S-PC/linux6.1_compile.md), [AIO-3588SG](../../核心板/Core-3588SG/linux6.1_compile.md) |

#### RK3576 系列

|  系统   |  板卡型号 |
|  ----  | ----  |
| Android14.0 | [ROC-RK3576-PC](../../主板/ROC-RK3576-PC/android_compile_android14.0_firmware.md#roc-rk3576-pc-chan-pin-bian-yi-fang-fa),[AIO-3576Q](../../核心板/iCore-3576Q/android_compile_android14.0_firmware.md#icore-3576q-chan-pin-bian-yi-fang-fa) |