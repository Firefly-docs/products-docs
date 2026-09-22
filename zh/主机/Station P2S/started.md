# 简介

[规格书](https://download.t-firefly.com/%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E6%96%87%E6%A1%A3/%E5%B0%8F%E5%9E%8B%E4%B8%BB%E6%9C%BA/Station%20P2S_Specification.pdf) | [购买链接](https://item.taobao.com/item.htm?spm=a1z10.5-c-s.w4002-24614730371.11.18435c47KlWMgK&id=687407527036) | [下载资料](https://community.t-firefly.com/doc/download/218)

Station P2S 采用 ROC-RK3568-PC-SE 四核64位 Cortex-A55 处理器，22nm 先进工艺，主频最高 2.0GHz，集成双核心架构 GPU 以及高效能 NPU（1TOPS）；最大支持 8G 大内存；支持 WiFi6、双千兆以太网；拥有丰富的接口扩展，支持多种视频输入输出接口，可适用于智能 NVR、云终端、物联网网关、工业控制、边缘计算等场景。

<center>

<img alt="" src="../../../rk356x_img/ROC-RK3568-PC-SE/Station-P2S.png" width="600">
</center>

## 接口描述

**Station P2S** 提供了丰富的接口，主要包括：

* 2 x RJ45（1000Mbps）
* 1 x HDMI 2.0（4K@60Hz）
* 1 x 控制串口（Control Port：RS232 × 2 + RS485 × 1）
* 1 x USB 3.0
* 2 x USB 2.0
* 1 x USB-C（OTG）
* 1 x TF Card 卡槽
* 1 x SIM 卡槽
* 1 x 3.5mm Audio Jack
* 1 x M.2 PCIe3.0 接口（NVMe SSD 2242）
* 1 x Mini PCIe 接口（4G 模块）
* 1 x SATA 3.0 接口（HDD/SSD）
* 1 x MIPI-CSI（30P-0.5mm）
* 1 x MIPI-DSI（30P-0.5mm）
* 1 x eDP（40P-0.5mm）
* 1 x 30P-2.0mm 扩展接口（ADC/USB2.0 HOST/I2C/I2S/PWM/UART/SPI/GPIO/SPEAKER/POWER KEY/RECOVERY）
* 1 x POE 接口（6P-2.0mm）
* 1 x Debug 调试串口（3P-2.0mm）
* Reset / Recovery / Power Key 按键

具体如下图：

<center>

<img alt="" src="../../../rk356x_img/ROC-RK3568-PC-SE/Station-P2S-interface_all.png" width="800">
</center>

## 结构尺寸

<center>

<img alt="" src="../../../rk356x_img/ROC-RK3568-PC-SE/Station-P2S-size.png" width="800">
</center>

## 资源与技术支持

* [[Wiki]](../../主板/ROC-RK3568-PC%20SE/index.md)：包含固件编译、系统使用、接口使用等教程(参考 ROC-RK3568-PC-SE Wiki)
* [[技术交流论坛]](https://forum.t-firefly.com/)：超过10万企业客户和用户沟通交流平台

### 联系方式

* 邮箱：sales@t-firefly.com
* 手机：(+86) 186 8811 7175
* 座机：0760-89881218
* 全国服务热线：4001-511-533
* 地址：广东省中山市东区中山四路 57 号宏宇大厦 2101 室
 <a id="firmware-format"></a>