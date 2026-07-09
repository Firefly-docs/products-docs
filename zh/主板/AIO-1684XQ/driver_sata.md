# SATA 使用
AIO-1684XQ 开发板上有 1 个 SATA3.0 接口:

![](../../../bm1684_img/AIO-1684XQ/sata.png)

硬件连接好 SATA 硬盘之后，待系统上电后，通过控制以下端口做到 SATA 设备的动态识别以及断开（需要 root 权限）：  
```
# 连接 SATA 设备
echo 1 >/sys/class/leds/SATA_EN/brightness
# 断开 SATA 设备
echo 0 >/sys/class/leds/SATA_EN/brightness
