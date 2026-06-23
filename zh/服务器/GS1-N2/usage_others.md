# 其他
## 蜂鸣器

|ON|OFF|
|---|---|
|echo 1 > /sys/class/leds/beeper_led/brightness|echo 0 > /sys/class/leds/beeper_led/brightness|

## LED 

||ON|OFF|
|---|---|---|
|电源|echo 1 > /sys/class/leds/ext_pwr_led1/brightness|echo 0 > /sys/class/leds/ext_pwr_led1/brightness|
|硬盘|echo 1 > /sys/class/leds/ext_sata_led2/brightness|echo 0 > /sys/class/leds/ext_sata_led2/brightness|
|网络|echo 1 > /sys/class/leds/ext_net_led3/brightness|echo 0 > /sys/class/leds/ext_net_led3/brightness|


## Power 
主模组给副模组提供**供电**，**开机**操作

||ON|OFF|
|---|---|---|
|供电|echo 1 > /sys/class/leds/res_core_en/brightness|echo 0 > /sys/class/leds/res_core_en/brightness|
|开机|echo 1 > /sys/class/leds/res_power/brightness  |echo 0 > /sys/class/leds/res_power/brightness|