# SATA 
AIO-1684XQ  has 1 SATA 3.0 interface on the development board:

![](../../../bm1684_img/AIO-1684XQ/sata.png)

After connecting the SATA hard drive to the hardware and powering on the system, you can use the following commands to dynamically detect and disconnect the SATA device (requires root permissions):
```
# Connect SATA device
echo 1 >/sys/class/leds/SATA_EN/brightness
# Disconnect SATA device
echo 0 >/sys/class/leds/SATA_EN/brightness
