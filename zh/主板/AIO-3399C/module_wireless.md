
# 无线模块
## [EC20 4G模组套件](https://store.t-firefly.com/goods.php?id=49)
### 产品参数
* **型号**
  * EC20-C R2.0 Mini PCIe-C
* **电源电压**
  * 3.3V~ 3.6V, 典型值： 3.3V
* **工作频段**
  * TDD-LTE: B38/B39/B40/B41
  * FDD-LTE: B1/B3/B8
  * WCDMA: B1/B8
  * TD-SCDMA: B34/B39
  * GSM: 900/1800 MHz
* **数据传输**
  * TDD-LTE： Max 130Mbps (DL) Max 35Mbps (UL)
  * FDD-LTE： Max 150Mbps (DL) Max 50Mbps (UL)
  * DC-HSPA+： Max 42Mbps (DL) Max 5.76Mbps (UL)
  * UMTS： Max 384Kbps (DL) Max 384Kbps (UL)
  * TD-SCDMA： Max 4.2Mbps (DL) Max 2.2Mbps (UL)
  * CDMA： Max 3.1Mbps (DL) Max 1.8Mbps (UL)
  * EDGE： Max 236.8Kbps (DL) Max 236.8Kbps (UL)
  * GPRS： Max 85.6Kbps (DL) Max 85.6Kbps (UL)
* **接口连接器**
  * USB：USB 2.0 高速接口, 480Mbps
  * 数字语音：1个数字语音接口 (可选)
  * USIM：1.8V/3V
  * 网络指示：×2, NET_STATUS 和 NET_MODE
  * UART：×1 UART
  * 复位：低电平
  * PWRKEY：低电平
  * 天线接口：×3 (主天线, 分集天线和GNSS天线接口)
  * ADC：×2
* **结构尺寸**
  * 51.0mm × 30.0mm × 4.9mm
* **重量**
  * 约 10.5g
* **认证**
  * CCC/ NAL*/ TA

### GNSS 功能

EC20 模组分带GNSS和不带GNSS两种，Firefly官网售卖的EC20 4G模组是不支持GNSS的，后缀为`SNNS`，支持GNSS功能的EC20 模组一般带有后缀`SGNS`，公版固件支持GNSS功能，但是默认关闭，使用方法可参考[EC20 GNSS功能](#ec20-gnss-gong-neng)章节。

### 实物图

![](../../../rk3399_img/module_wireless_ec20.png)

### 连接方法


* USB 接口连接

![](../../../rk3399_img/AIO-3399C/module_wireless_usb.jpg)



* Mini-PCIe 接口连接

![](../../../rk3399_img/AIO-3399C/module_wireless_mini-pcie.jpg)

* SIM 卡连接

![](../../../rk3399_img/module_wireless_sim_connection.jpg)


### 参考固件

公版固件默认支持 EC20 4G 模组 [[链接]](https://www.t-firefly.com/doc/download/134.html)

## EC200T 4G模组套件

### 产品参数
* **型号**
  * EC200T-CN Mini PCIe-D
* **电源电压**
  * 3.4V~ 4.3V, 典型值： 3.8V
* **工作频段**
  * TDD-LTE: B34/B38/B39/B40/B41
  * FDD-LTE: B1/B3/B5/B8
  * WCDMA: B1/B5/B8
  * GSM: 900/1800 MHz
* **数据传输**
  * TDD-LTE： Max 120Mbps (DL) Max 3Mbps (UL)
  * FDD-LTE： Max 150Mbps (DL) Max 50Mbps (UL)
  * DC-HSDPA： Max 21Mbps (DL)
  * HSDPA: Max 5.76 Mbps (UL)
  * WCDMA: Max 384Kbps (DL) Max 384Kbps (UL)
  * EDGE: Max 236.8Kbps (DL) Max 236.8Kbps (UL)
  * GPRS： Max 85.6Kbps (DL) Max 85.6Kbps (UL)
* **接口连接器**
  * USB：USB 2.0 高速接口, 480Mbps
  * 数字语音：1个数字语音接口 (可选)
  * USIM：1.8V/3V
  * UART：×1 UART
  * UART_DTR (睡眠控制)
  * W_DISABLE# (飞行模式控制)
  * LED_WWAN# (网络状态指示)
  * PERST# (模块复位)
  * RI (主机唤醒)
  * WAKEUP_IN (睡眠控制,低电平有效)
  * WAKEUP_OUT (睡眠状态指示)
  * 天线接口：×2 (主天线, 分集天线)
* **结构尺寸**
  * 51.0mm × 30.0mm × 4.9mm
* **重量**
  * 约 10.2g
* **认证**
  * CCC/SRRC/NAL

### 实物图

![](../../../rk3399_img/module_wireless_ec200t.jpg)

### 连接方法

参考[EC20 4G模组套件](#ec20-4g-mo-zu-tao-jian)

### 参考固件

公版固件默认支持 EC200 4G 模组 [[链接]](https://www.t-firefly.com/doc/download/139.html)








## GNSS 模块

### [DELINCOMM 系列](https://store.t-firefly.com/goods.php?id=43)

#### 产品参数

![](../../../rk3399_img/module_wireless_gps_delincomm.jpg)

该系列各个模组型号的参数会存在差异，详情可参考[DelinComm_gps.pdf](http://download.t-firefly.com/product/Board/Common/Peripheral/Wireless/GPS%E6%A8%A1%E5%9D%97/DelinComm_gps.pdf)

#### 接口定义

VCC (红线)、GND (黑线)、TX (白线)、RX(绿线)

#### 使用说明

##### <font size=3>硬件连接</font>

* 将模组的VCC、GND、TX、RX分别接到 AIO-3399C **UART2**(<font color = "red">对应节点为`/dev/ttysWK2`</font>)的3.3V、GND、RX、TX，注意避免接错VCC、GND、TX、RX<font color = "red">导致烧坏模组</font>
* 对于UART的一些定义和说明可以参考Wiki教程[《UART 使用》](driver_uart.html)

##### <font size=3>软件配置</font>

DELINCOMM系列串口波特率配置为`9600`，公版固件已经默认配置好，使能GPS后即可使用，详情可以参考[如何使能GPS和修改串口配置](#ru-he-shi-neng-gps-he-xiu-gai-chuan-kou-pei-zhi)。

**注意**:GPS功能会占用到 **UART2**，若需要用 **UART2** 作为其他用途，需要将GPS先禁止掉

#### 参考固件

公版新固件默认支持GPS模组, 但需要手动打开。 或直接下载GPS默认启动固件[百度云](https://www.t-firefly.com/doc/download/135.html)

### DK2635U7F 模块

#### 产品参数

![](../../../rk3399_img/module_wireless_gps.png)

详细产品规格可参考[DK2635U7F规格书.pdf](http://download.t-firefly.com/product/Board/Common/Peripheral/Wireless/GPS%E6%A8%A1%E5%9D%97/DK2635U7F%E8%A7%84%E6%A0%BC%E4%B9%A6.pdf)

#### 接口定义

VCC (白线)、GND (黑线)、TX (蓝线)、RX(绿线)

#### 使用说明

##### <font size=3>硬件连接</font>

* 将模组的VCC、GND、TX、RX分别接到 AIO-3399C **UART2**(<font color = "red">对应节点为`/dev/ttysWK2`</font>)的3.3V、GND、RX、TX，注意避免接错VCC、GND、TX、RX<font color = "red">导致烧坏模组</font>
* 对于UART的一些定义和说明可以参考Wiki教程[《UART 使用》](driver_uart.html)

##### <font size=3>软件配置</font>

UBLOX模块串口波特率配置为`9600`，公版固件已经默认配置好，使能GPS后即可使用，详情可以参考[如何使能GPS和修改串口配置](#ru-he-shi-neng-gps-he-xiu-gai-chuan-kou-pei-zhi)。

**注意**:GPS功能会占用到 **UART2**，若需要用 **UART2** 作为其他用途，需要将GPS先禁止掉

#### 参考固件

公版新固件默认支持GPS模组, 但需要手动打开。 或直接下载GPS默认启动固件[百度云](https://www.t-firefly.com/doc/download/135.html)


### EC20 GNSS功能

EC20模组支持GPS、GLONASS、GALILEO、BEIDOU，兼容标准 NMEA 0183 协议，可通过 USB NMEA 接口输出 1Hz 频率的 NMEA 信息，默认输出串口为`/dev/ttyUSB1`，波特率`115200` bit/s。

#### 天线要求
* 频率范围:1559MHz~1609MHz
* 极化:RHCP 或 Linear
* VSWR:< 2(典型值)
* 有源天线噪声系数:< 1.5dB
* 有源天线增益:> 0dB
* 有源天线内嵌 LNA 增益:< 17dB

#### 使用说明

##### <font size=3>硬件连接</font>

EC20模组根据[EC20 4G模组套件](#ec20-4g-mo-zu-tao-jian)章节的说明连接，天线接入带有GPS丝印的天线接口。

##### <font size=3>软件配置</font>

对于EC20模组，其GNSS 配置的串口节点和波特率分别为`/dev/ttyUSB1`、`115200`，使用模组之前需要检查`/system/etc/u-blox.conf`中`SERIAL_DEVICE`或者`SERIAL_BAUD_RATE`的值是否与之匹配，确认匹配则使能GPS后即可使用，使能方法参考[如何使能GPS和修改串口配置](#ru-he-shi-neng-gps-he-xiu-gai-chuan-kou-pei-zhi)。

**注意**:对于EC20模组，使用GNSS功能时，需要注意是否有接入其他的`ttyUSB*`设备，存在其他`ttyUSB`时，有可能会使EC20的默认`ttyUSB`设备号发生变化，导致GNSS功能无法使用

### 如何使能GPS和修改串口配置

名词解释:
* `SERIAL_DEVICE`: 串口节点
* `SERIAL_BAUD_RATE`: 串口波特率

##### 临时修改

确保固件为最新[公版固件](https://www.t-firefly.com/doc/download/54.html)，固件默认关闭GPS功能，需要手动使能，以下是修改方法：

* 使能ADB，如何使能ADB参考Wiki教程[《ADB的使用》](adb_use.html)
* 设置系统可读可写
    ```
    adb shell setprop persist.sys.root_access 3
    adb root && adb remount
    ```
* 修改参数

    * 使能GPS：修改机器上`/system/build.prop`(Android7.1以上的版本为`/vendor/build.prop`)里面`ro.factory.hasGPS`参数为`true`，实现使能GPS，软重启机器后生效。
    * 修改串口配置(串口节点或者波特率)：将机器上`/system/etc/u-blox.conf`里面`SERIAL_DEVICE`或者`SERIAL_BAUD_RATE`修改成对应模组配置的值，软重启机器后生效。

##### 代码修改

* 使能GPS
    * 修改SDK目录下`device/rockchip/rk3399/rk3399_firefly_aioc.mk`(Android7.1以上的版本为`device/rockchip/rk3399/rk3399_firefly_aioc/rk3399_firefly_aioc.mk`)里面`BOARD_HAS_GPS`参数为`true`，实现固件使能GPS功能，修改后重新编译SDK并烧录固件生效。

* 修改串口配置(串口节点或者波特率)
    * 将SDK目录下`device/rockchip/rk3399/gps/u-blox.conf`(Android7.1以上的版本为`device/rockchip/rk3399/rk3399_firefly_aioc/gps/u-blox.conf`)里面`SERIAL_DEVICE`或者`SERIAL_BAUD_RATE`修改成对应模组配置的值，修改后重新编译SDK并烧录固件生效

