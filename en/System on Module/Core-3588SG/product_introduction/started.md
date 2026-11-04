[Specification](https://download.t-firefly.com/%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E6%96%87%E6%A1%A3/%E6%A0%B8%E5%BF%83%E6%9D%BF/Core-3588SG_Octa%20Core%208K%20AI%20Core%20Board_Specification.pdf?v=1704419444) | [Downloads](https://community.t-firefly.com/en/doc/download/201)

# Introduction

Based on Rockchip new generation of flagship AIOT chip -- RK3588S, the
[Core-3588SJD4](https://item.taobao.com/item.htm?id=676534397877) features an 8nm LP process and an 8-core (Cortex-A76x4+
Cortex-A55x4) 64-bit CPU with up to 2.4 GHz.

Integrated ARM Mali-G610MP4 quad-core GPU, built-in AI accelerator NPU,
can provide 6Tops computing power, support the mains, support up to 32 GB of memory, support WiFi 6, 5G/4G and other
high-speed wireless network communication, support 8K video codec and
multiple formats of video input and output, support multiple operating
systems can be applied to ARM PC, edge computing, cloud server,
intelligent NVR and other fields.

<center>

<img alt="" src="../../../../rk3588_img/Core-3588SG/Core-3588SG-B.png" width="600">
</center>

The AIO-3588SG development board consists of the core board **Core-3588SG** + **MB-G-
RK3588S**. AIO-3588SG
has rich interfaces such as RGMII, CAN, USB3.0, I2C,
SPI, UART, GPIO, MIPI-DSI, and MIPI-CSI, provide multiple power supply
modes. It can be directly applied to various intelligent products to
accelerate product implementation. For details, refer to ["interface
definition"](../interface_definition.md).


<center>

<img alt="" src="../../../../rk3588_img/Core-3588SG/AIO-3588SG-B-1.png" width="600">
</center>

### The AIO-3588SG standard kit contains the following accessories (for reference only): 

-   Core-3588SG Core Board x 1-   MB-JD4-RK3588S  x 1
-   Copper tube antenna x 3
-   Type-C data cable x 1

Optional accessories include:

-   Serial port module of Firefly

In addition, you may need the following accessories during use:

-   Display device
    -   101-M101014-BE45-A1 Display panel + adapter panel + 40pin extension cable
-   Network
    -   100M/1000M Ethernet cable and wired router
    -   WiFi router
-   Input device
    -   USB wireless/wired mouse/keyboard
-   Upgrade firmware, debug
    -   Type-C data cable
    -   Serial to USB adapter 

## Things to know before use
### power supply

AIO-3588SG is powered by battery and TYPE-C: 
* Use the battery to supply power, which is the same as the mobile phone. When the battery is dead, you need to plug in TYPE-C to charge. For battery specifications, refer to the Battery chapter used by the interface.
* Use TYPE-C charging head for power supply. 9V3A or 12V2A PD charging head is recommended. Ordinary 5V charger does not meet the startup requirements of development board.

### display

The AIO-3588SG does not have an HDMI display port, which can be displayed in two ways: 
* In case of screen projection through TYPE-C or DP display using TYPE-C interface, battery needs to be connected;
* It is recommended to use the official mipi screen for display. You need to consult the after-sales service for purchase. This method doesn't care if the battery is connected.

### ADB

AIO-3588SG can be connected to ADB in two ways:
* Use OTG to access ADB, OTG interface is TYPE-C, that is to say, when using ADB can not use TYPE-C power supply, at this time need to connect to the battery.
* Use network ADB, which doesn't care if the battery is connected.

## Interface Description

AIO-3588SG provides rich interfaces, including:

* 1 x Display Port1.4 (Max output support 8K@30Hz)
* 1 x USB3.0 OTG(Type-C)
* 2 x MIPI-DSI
* 1 x EDP
* 1 x MIPI-CSI
* 1 x MIPI-D/CPHY-CSI
* 1 x TF Card
* 1 x USB3.0
* 1 x USB2.0(One USB2.0 are led by a 40-pin pin row)
* 1 x WIFI (Support wifi6)
* 1 x Bluetooth
* 1 x FAN
* 2 x RJ45 (Suport 1Gbps)
* 1 x Speaker


The details are shown below：

<center>

<img alt="" src="../../../../rk3588_img/Core-3588SG/interface_all-B.png" width="900">
</center>


## Resources and Support

* [[Technical Forum]](https://forum.t-firefly.com/): A communication platform with more than 100,000 enterprise customers and users

### Contact

* Email: sales@t-firefly.com
* Mobile: (+86) 186 8811 7175
* Landline: 0760-89881218
* Address: Room 2101, Hongyu Building, No. 57 Zhongshan 4th Road, East District, Zhongshan City, Guangdong Province