# 其他
## 蜂鸣器

|ON|OFF|
|---|---|
|echo 1 > /sys/class/leds/:beeper_led/brightness|echo 0 > /sys/class/leds/:beeper_led/brightness|

## LED 

||ON|OFF|
|---|---|---|
|电源|echo 1 > /sys/class/leds/:work/brightness|echo 0 > /sys/class/leds/:work/brightness|
|硬盘|echo 1 > /sys/class/leds/:sata_led/brightness|echo 0 > /sys/class/leds/:sata_led/brightness|
|网络|echo 1 > /sys/class/leds/:user/brightness|echo 0 > /sys/class/leds/:user/brightness|