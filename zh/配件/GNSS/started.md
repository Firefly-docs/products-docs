# 产品介绍

## GPS、GLONASS 与 RG4538 定位模块

Firefly 定位模块支持 GPS、GLONASS 或北斗卫星定位，使用 UART/TTL 接口与主板连接。

![](../../../modules_img/GNSS/gnss_RG4538_en.png)

## 规格参数

| 型号 | 芯片 | 协议 | 波特率 |
| ---- | ---- | ---- | ---- |
| RG4538 | G9501 | NMEA-0183 | 9600 |
| RM4538-B | MT3333 | NMEA-0183 | 9600 |
| RU4538-G | UBX-M8030-KT | NMEA-0183 | 9600 |
| DK2635U7F | UBX-G7020-KT | NMEA-0183 | 9600 |

## 接线说明

将模块的 VCC、GND、TX 和 RX 分别连接至主板的 3.3V、GND、RX 和 TX。接线错误可能损坏模块。

## 使用说明

在 Android 系统中使用定位模块前，需要在 `/vendor/build.prop` 中将 `ro.factory.hasGPS` 设为 `true`，并在 `/system/etc/u-blox.conf` 中配置模块对应的串口路径和波特率。修改后重启设备生效。

相关文档和固件请查看官网[资料下载](https://community.t-firefly.com/doc/download/120)。