# Wireless module

## SLM630B 4G module

### Product parameter

* Model
    * SLM630B
* Power supply
    * 3.3V～4.2V (Typical 3.3V)
* Band
    * TD-LTE: Band38/39/40/41
    * FDD-LTE： B1/3
    * WCDMA: B1
    * TD-SCDMA: B34/39
    * EVDO: BC0
    * CDMA 1x: BC0
    * GSM: Band 8/3
* Data
    * TD-LTE： Rel 9 Cat3 TD-LTE 61Mbps/18Mbps
    * FDD-LTE： Rel 9 Cat3 FDD-LTE 100Mbps/50Mbps
    * DC-HSPA+： 42Mbps/5.76Mbps
    * HSPA： 7.2Mbps/5.76Mbps
    * WCDMA： 384kbps/384kbps
    * EVDO RevB： 14.7Mbps/5.4Mbps
    * EVDO RevA： 3.1Mbps/1.8Mbps
    * TD-HSPA： 4.2Mbps/2.2Mbps
    * TD-SCDMA： 384kbps/384kbps
    * CDMA： 1x 153.6kbps/153.6kbps
    * EDGE： 236.8kbps/236.8kbps
    * GPRS： 85.6kbps/85.6kbps
    * CSD： GSM CSD: 14.4kbps
* TCP/IP：Embedded TCP/IP protocol stack
* Connector
    * PCI express Mini Card connector
    * HRS U.FL-R-SMT-1 RF connector ×3
* Dimensions
    * 51.0×30.0×4.8mm
* Approvals
    * RoHS  CCC
* Related technical information：[SLM630B](https://pan.baidu.com/s/1eSld2gU#list/path=%2FPublic%2FDevBoard%2FFirefly-RK3399%2FFirmware%2FPeripherals%2F4Gdongle%2F%E7%BE%8E%E6%A0%BCSLM630B%E6%A8%A1%E5%9D%97%2F%E8%B5%84%E6%96%99&parentPath=%2FPublic%2FDevBoard%2FFirefly-RK3399%2FFirmware%2FPeripherals%2F4Gdongle%2F%E7%BE%8E%E6%A0%BCSLM630B%E6%A8%A1%E5%9D%97)

### Firmware follow

<font color=#ff0000>SLM630B USB 4G module firmware link:</font>[firmware](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)

<font color=#ff0000>SLM630B USB 4G module patch link:</font>[patch](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)

### Picture

![](../../img/Firefly-RK3399/module_wireless_slm630b.en.jpg)

### Connection Method

* USB connection

![](../../img/Firefly-RK3399/module_wireless_aerial.jpg)

![](../../img/Firefly-RK3399/module_wireless_usb.png)

* Mini-PCIe connection

![](../../img/Firefly-RK3399/module_wireless_mini-pcie.en.jpg)

##  [EC20 4G module](https://www.firefly.store/products/4g-module-kit-eg25-g)

### Product Parameter

* Model
    * EC20-C R2.0 Mini PCIe-C
* Power supply
    * 3.3V～3.6V (Typical 3.3V)
* Band
    * TDD-LTE: B38/B39/B40/B41
    * FDD-LTE：B1/B3/B8
    * WCDMA: B1/B8
    * TD-SCDMA: B34/B39
    * GSM: 900/1800
* Data
    * TDD-LTE： Max 130Mbps (DL) Max 35Mbps (UL)
    * FDD-LTE： Max 150Mbps (DL) Max 50Mbps (UL)
    * DC-HSPA+： Max 42Mbps (DL) Max 5.76Mbps (UL)
    * UMTS： Max 384Kbps (DL) Max 384Kbps (UL)
    * CDMA： Max 3.1Mbps (DL) Max 1.8Mbps (UL)
    * TD-SCDMA： Max 4.2Mbps (DL) Max 2.2Mbps (UL)
    * EDGE： Max 236.8Kbps (DL) Max 236.8Kbps (UL)
    * GPRS： Max 85.6Kbps (DL) Max 85.6Kbps (UL)
* Connector
    * USB: USB 2.0 high-speed interface, 480Mbps
    * Digital voice: 1 digital voice interface (optional)
    * USIM: 1.8 V / 3 V
    * Network instructions: * 2, NET_STATUS, and NET_MODE
    * UART: x 1 UART
    * Reset: low level
    * PWRKEY: low level
    * Antenna interface: x 3 (main antenna, split antenna and GNSS antenna interface)
    * ADC: x 2
* Dimensions
    * 51.0×30.0×4.9mm
* Weight
    * About 10.5 g
* Approvals
    * CCC/ NAL*/ TA*

### Firmware follow

<font color=#ff0000>The official website of the public version of the default firmware support EC20 4Gdongle module, Baidu Pan link：</font>[public version of the default firmware](http://www.t-firefly.com/share/index/listpath/id/ab0b19d49d35105ec574a2caf307103e.html)

<font color=#ff0000>The EC20 USB 4G module patch link:</font>[patch](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)

## [GPS module](https://www.firefly.store/products/gps-module)

### Product Parameter

![](../../img/module_wireless_gps.en.jpg)

### Firmware download

The firmware defaults to open the GPS.

### Modification method

GPS is available by default in firmware above, but the public version firmware defaults to close it. You can modify `ro.factory.hasGPS` in `/system/build.prop` to close or open GPS. Then, please restart your board after completing the modification.

### 3.Note

The GPS function will occupy uart4, so please follow above modification method to close the GPS before using uart4 to do other things.