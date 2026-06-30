
<font color=red>**注意：** 4G模组和5G模组使用的座子不一样，4G模组使用MINI PCIE接口，5G模组使用M.2(NGFF)接口，购买AIO-3568J主板时请根据所需的模组选择正确的接口座子</font>

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

![](../../../rk356x_img/module_wireless_ec20.png)

### 连接方法

* USB 接口连接

![](../../../rk356x_img/module_wireless_ec20_usb.jpg)


* SIM 卡连接

![](../../../rk356x_img/module_wireless_sim_connection.jpg)

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

![](../../../rk356x_img/module_wireless_ec200t.jpg)

### 连接方法

参考[EC20 4G模组套件](#ec20-4g-mo-zu-tao-jian)


### 参考固件

公版固件默认支持 EC20 4G 模组、EC200T 4G模组 






## GNSS 模块


### EC20 GNSS功能

EC20模组支持GPS、GLONASS、GALILEO、BEIDOU，兼容标准 NMEA 0183 协议，可通过 USB NMEA 接口输出 1Hz 频率的 NMEA 信息，默认输出串口为`/dev/ttyUSB1`，波特率`115200` bit/s。

#### 天线要求
* 频率范围:1559MHz~1609MHz
* 极化:RHCP 或 Linear
* VSWR:< 2(典型值)
* 有源天线噪声系数:< 1.5dB
* 有源天线增益:> 0dB
* 有源天线内嵌 LNA 增益:< 17dB

**注意:** GPS天线需要使用**有源天线**

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

确保固件为最新[公版固件]()，固件默认关闭GPS功能，需要手动使能，以下是修改方法：

* 使能ADB，如何使能ADB参考Wiki教程[《ADB的使用》](adb_use.html)
* 设置系统可读可写
    ```
    adb shell setprop persist.sys.root_access 3
    adb root && adb remount
    ```
* 修改参数

    * 使能GPS：修改机器上`/vendor/build.prop`里面`ro.factory.hasGPS`参数为`true`，实现使能GPS，软重启机器后生效。
    * 修改串口配置(串口节点或者波特率)：将机器上`/system/etc/u-blox.conf`里面`SERIAL_DEVICE`或者`SERIAL_BAUD_RATE`修改成对应模组配置的值，软重启机器后生效。

##### 代码修改

* 使能GPS
    * 修改SDK目录下`device/rockchip/rk356x//.mk`里面`BOARD_HAS_GPS`参数为`true`，实现固件使能GPS功能，修改后重新编译SDK并烧录固件生效。

* 修改串口配置(串口节点或者波特率)
    * 将SDK目录下`device/rockchip/rk356x//gps/u-blox.conf`里面`SERIAL_DEVICE`或者`SERIAL_BAUD_RATE`修改成对应模组配置的值，修改后重新编译SDK并烧录固件生效