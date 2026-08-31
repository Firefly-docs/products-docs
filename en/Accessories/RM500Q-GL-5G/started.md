# 一、Introduction
## Product introduction
### RM500Q-GL

![](../../../modules_img/RM500Q-GL-5G/5G.png)

## Detailed parameters

* **Model**
  * RM500QGLAB-M20-SGASA
* **Supply voltage**
  * 3.135~4.4 V , Typical values: 3.7V
* **Transmitting power**
  * WCDMA band：Class 3（24 dBm +1/-3 dB）
  * LTE band：Class 3（23 dBm ±2 dB）
  * 5G NR band：Class 3（23 dBm ±2 dB）
  * LTE B38/B40/B41/B42/B43 band HPUE：Class 2（26 dBm ±2 dB）
  * 5G NR n41/n77/n78/n79 bands HPUE：Class 2（26 dBm +2/-3 dB)
* **5G NR characteristics**
  * support 3GPP Release 15
  * modulation mode：
    * 256QAM (UL)
    * 256QAM (DL)
  * SCS supports 15 kHz and 30 kHz
  * the maximum bandwidth of 5g NR band under development supports 20MHz
  * 5G NR n41/n77/n78/n79 , the maximum bandwidth is 100 MHz
  * support Option 3x、3a and Option 2
  * NSA TDD：Max 2.5 Gbps (DL) Max 650 Mbps (UL)
  * SA TDD： Max 2.1 Gbps (DL) Max 900 Mbps (UL)
  * n41/n77/n78/n79 bands support 2 × 2 MIMO (UL)
  * n1/n2/n3/n7/25/n38/n40/n41/n48/n66/n77/n78/n79 bands support 4 × 4 MIMO (DL)
  * support SA and NSA networking mode, all 5G bands support SA，n38*/n41/n77/n78/n79 bands support NSA
* **LTE characteristics**
  * support CA Cat 16 FDD and TDD
  * modulation mode：
    * (UL) QPSK、16QAM、64QAM、256QAM*
    * (DL) QPSK、16QAM、64QAM、256QAM
  * support 1.4~20 MHz（3 × CA）RF bandwidth
  * LTE： Max 1.0 Gbps (DL) Max 200 Mbps (UL)
  * B1/B2/B3/B4/B7/B25/B30/B38/B39/B40/B41/B42/B43/B48/B66 bands support (DL) 4 × 4 MIMO
* **UMTS characteristics**
  * support QPSK、16QAM and 64QAM modulation
  * DC-HSDPA：Max 42 Mbps (DL)
  * HSUPA：Max 5.76 Mbps (UL)
  * WCDMA：Max (DL/UL) 384 kbps
  * support 3GPP R8 DC-HSDPA、HSPA+、HSDPA、HSUPA and WCDMA
* **Interface connector**
  * USB interface： USB 3.1 and USB 2.0 standard
  * (U)SIM interface：support Class B（3.0 V）and Class C（1.8 V）
  * diversity receiving antenna interface ：support 5G NR/LTE/WCDMA diversity receiving
  * PCIE interface：PCIe Gen3 standard, up to 8 Gbps per channel
  * antenna interface：×4 (ANT0、ANT1、ANT2_GNSSL1 and ANT3)
* **Characteristics of network protocol**
  * support QMI/NTP* protocol
  * support PAP and EIRP protocol, commonly used for PPP connections
* **Built-in GNSS**
  * support GPS, GLONASS, BeiDou, Galileo and QZSS
* **Structure size**
  * (52.0 ±0.15) mm × (30.0 ±0.15) mm × (2.3 ±0.2) mm
* **Weight**
  * about 8.6g
* **RoHS**
  * EU RoHS standard

**Note：**`*` indicates that Quectel is under development


# 二、Usage

<!--
## 使用说明
-->

## Hardware connection
### Module connection

<!-- | CPU | Board | 
| ---- | ---- | 
| RK3568 | [AIO-3568J](_images/5G_AIO-3568J.png) | 
| RK3588 | [ITX-3588J](_images/5G_ITX-3588J.jpg), [AIO-3588SJD4](_images/5G_AIO-3588SJD4.jpg),[AIO-3588Q](_images/5G_AIO-3588Q.jpg)| -->

![](../../../modules_img/RM500Q-GL-5G/5G_PCIE.png)


### SIM insertion
![](../../../modules_img/RM500Q-GL-5G/ec20_sim.png)

# 三、Firmware and Resource download
Related documents and firmware download, see the official website [Resource Download](https://community.t-firefly.com/en/doc/download/118)。

<!--
## 文档下载
## 配件图纸
## 软件工具
## 固件下载
-->

# 四、Tutorial
## Flash firmware

| CPU | USB upgrade | SD upgrade |
| ---- | ---- | ---- |
|RK3568|[AIO-3568J](../../Motherboard/AIO-3568J/03-upgrade_firmware.md) | [AIO-3568J](../../Motherboard/AIO-3568J/05-upgrade_firmware_sd.md)| 
|RK3588|[ITX-3588J](../../Motherboard/ITX-3588J/upgrade_firmware.md), [AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/upgrade_firmware.md) ,[AIO-3588Q](../../Motherboard/AIO-3588Q/upgrade_firmware.md)| [ITX-3588J](../../Motherboard/ITX-3588J/upgrade_firmware_sd.md), [AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/upgrade_firmware_sd.md) ,[AIO-3588Q](../../Motherboard/AIO-3588Q/upgrade_firmware_sd.md)|

<!-- ## Compile the firmware

### RK3568 platform

|  System   |  Board | 
|  ----  | ----  | 
| Android11.0 | [AIO-3568J](../../Motherboard/AIO-3568J/compile_android11.0_firmware.md)
| Ubuntu | [AIO-3568J](../../Motherboard/AIO-3568J/ubuntu_compile.md) | 
| Buildroot | [AIO-3568J](../../Motherboard/AIO-3568J/buildroot_compile.md)

### RK3588 系列

|  系统   |  板卡型号 | 
|  ----  | ----  | 
| Android12.0 | [ITX-3588J](../../Motherboard/ITX-3588J/android_compile_android12.0_firmware.md) , [AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/android_compile_android12.0_firmware.md),[AIO-3588Q](../../Motherboard/AIO-3588Q/android_compile_android12.0_firmware.md)|
| Buildroot | [ITX-3588J](../../Motherboard/ITX-3588J/linux_compile_buildroot.md),[AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/linux_compile_buildroot.md),[AIO-3588Q](../../Motherboard/AIO-3588Q/linux_compile_buildroot.md),[AIO-3588MQ](../../Motherboard/AIO-3588MQ/linux_compile_buildroot.md),[AIO-3588JQ](../../Motherboard/AIO-3588JQ/linux_compile_buildroot.md)|
| Ubuntu20.04 | [ITX-3588J](../../Motherboard/ITX-3588J/linux_compile_ubuntu.md),[AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/linux_compile_ubuntu.md),[AIO-3588Q](../../Motherboard/AIO-3588Q/linux_compile_ubuntu.md),[AIO-3588MQ](../../Motherboard/AIO-3588MQ/linux_compile_ubuntu.md),[AIO-3588JQ](../../Motherboard/AIO-3588JQ/linux_compile_buildroot.md)|
| Debian11 | [ITX-3588J](../../Motherboard/ITX-3588J/linux_compile_debian.md),[AIO-3588SJD4](../../Motherboard/AIO-3588SJD4/linux_compile_debian.md),[AIO-3588Q](../../Motherboard/AIO-3588Q/linux_compile_debian.md),[AIO-3588MQ](../../Motherboard/AIO-3588MQ/linux_compile_debian.md),[AIO-3588JQ](../../Motherboard/AIO-3588JQ/linux_compile_debian.md)| -->


## GNSS Function(optional)
GNSS is supported by public firmware, but disabled by default.

### Parameters
Supports GPS, GLONASS, GALILEO and BEIDOU, and is compatible with standard NMEA 0183 protocol. It can output NMEA information of 1Hz frequency through USB NMEA interface. The default output serial port is /dev/ttyUSB1 and baud rate is 115200 bit/s.

### Antenna Requirements

* Frequency range: 1559MHz~1609MHz
* Polarization: RHCP or Linear
* VSWR: < 2(Typical)
* Passive antenna gain: > 0dBi

### How to enable GPS and modify serial port Configuration
[Reference EC20](../EC20/started.md#how-to-enable-gps-and-modify-serial-port-configuration)