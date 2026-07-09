# 无线模块

## [EC20 4G模组套件](https://store.t-firefly.com/goods.php?id=49)

### 产品参数

* 型号
    + EC20-C R2.0 Mini PCIe-C
* 电源电压
    + 3.3V~ 3.6V, 典型值： 3.3V
* 工作频段
    + TDD-LTE: B38/B39/B40/B41
    + FDD-LTE: B1/B3/B8
    + WCDMA: B1/B8
    + TD-SCDMA: B34/B39
    + GSM: 900/1800
* 数据传输
    + TDD-LTE： Max 130Mbps (DL) Max 35Mbps (UL)
    + FDD-LTE： Max 150Mbps (DL) Max 50Mbps (UL)
    + DC-HSPA+： Max 42Mbps (DL) Max 5.76Mbps (UL)
    + UMTS： Max 384Kbps (DL) Max 384Kbps (UL)
    + TD-SCDMA： Max 4.2Mbps (DL) Max 2.2Mbps (UL)
    + CDMA： Max 3.1Mbps (DL) Max 1.8Mbps (UL)
    + EDGE： Max 236.8Kbps (DL) Max 236.8Kbps (UL)
    + GPRS： Max 85.6Kbps (DL) Max 85.6Kbps (UL)
* 接口连接器
    + USB：USB 2.0 高速接口, 480Mbps
    + 数字语音：1个数字语音接口 (可选)
    + USIM：1.8V/3V
    + 网络指示：×2, NET_STATUS 和 NET_MODE
    + UART：×1 UART
    + 复位：低电平
    + PWRKEY：低电平
    + 天线接口：×3 (主天线, 分集天线和GNSS天线接口)
    + ADC：×2
* 结构尺寸
    + 51.0mm × 30.0mm × 4.9mm
* 重量
    + 约 10.5g
* 认证
    + CCC/ NAL*/ TA

### 实物图

![](../../../rk3288_img/module_wireless_ec20_1.png)

### 连接方法

* USB接口连接

![](../../../rk3288_img/AIO-3288J/module_wireless_ec20_2.png)


* Mini-PCIe接口连接

![](../../../rk3288_img/AIO-3288J/module_wireless_ec20_3.png)


### 参考固件

公版固件默认支持 EC20 4G 模组。






## GNSS 模块

### [DELINCOMM 系列](https://store.t-firefly.com/goods.php?id=43)

#### 产品参数

![](../../../rk3288_img/module_wireless_gps_delincomm.jpg)

该系列各个模组型号的参数会存在差异，详情可参考[DelinComm_gps.pdf](http://download.t-firefly.com/product/Board/Common/Peripheral/Wireless/GPS%E6%A8%A1%E5%9D%97/DelinComm_gps.pdf)

#### 连接方法

对于 DELICOMM 系列的GPS模块，需用到 VCC (红线)，GND (黑线)，TX (白线)，分别接到 AIO-3288J UART3对应的 3.3V、GND、RX 这三个地方。

### DK2635U7F 模块

#### 产品参数

![](../../../rk3288_img/module_wireless_ublox.png)

详细产品规格可参考[DK2635U7F规格书.pdf](http://download.t-firefly.com/product/Board/Common/Peripheral/Wireless/GPS%E6%A8%A1%E5%9D%97/DK2635U7F%E8%A7%84%E6%A0%BC%E4%B9%A6.pdf)

#### 连接方法

对于 UBLOX 模块，同样需用到 VCC (白线)，GND (黑线)，TX (蓝线)，可以分别接到 AIO-3288J UART3对应的 3.3V、GND、RX 这三个地方。


### 修改方法


```
 --- a/device/rockchip/rk3288/gps/u-blox.conf
 +++ b/device/rockchip/rk3288/gps/u-blox.conf
 @@ -35,7 +35,7 @@

  ### Serial interface
  # Device to be used by the driver
 -SERIAL_DEVICE          /dev/ttyS4
 +SERIAL_DEVICE          /dev/ttysWK2
  #SERIAL_DEVICE         /dev/s3c2410_serial2
  #SERIAL_DEVICE         /dev/i2c-4
  #SERIAL_DEVICE         /dev/ttyACM0
```

### 参考固件

公版固件默认关闭该功能,用户可以根据需要修改 `/system/build.prop`里面 `ro.factory.hasGPS` 参数 (false/true)，实现禁止或者使能 GPS 功能，改完之后重启机器使之生效。

### 注意事项

GPS功能会占用到UART3，若需要用UART3作为其他用途，需要将GPS的功能先禁止掉。

