# 蜂鸣器

|ON|OFF|
|---|---|
|echo 1 > /sys/class/leds/beeper_led/brightness|echo 0 > /sys/class/leds/beeper_led/brightness|

# LED 

||ON|OFF|
|---|---|---|
|电源|echo 1 > /sys/class/leds/ext_pwr_led1/brightness|echo 0 > /sys/class/leds/ext_pwr_led1/brightness|
|硬盘|echo 1 > /sys/class/leds/ext_sata_led2/brightness|echo 0 > /sys/class/leds/ext_sata_led2/brightness|
|网络|echo 1 > /sys/class/leds/ext_net_led3/brightness|echo 0 > /sys/class/leds/ext_net_led3/brightness|


# 主板上电模式

* 主板自动上电，需要把拨码开关 `AUTO_PWR` 拨到 `ON`
* 主板按POWER键上电，需要把拨码开关 `AUTO_PWR` 拨到 `1`

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/auto_pwr.png)

# RK182X 电压选择
需要把拨码开关 `RES_VOL_SEL` 拨到 `ON`
![](../../../gs1-n2_img/AIO-GS1N2-RK182X/res_vol_sel.png)

# USB
host 模式时，需要把拨码开关 `USB SEL` 拨到 `ON`
![](../../../gs1-n2_img/AIO-GS1N2-RK182X/usb_sel.png)

# 风扇
## RK1820/RK1828 SODIMM 模组
热管理策略为当前的策略说明，后续持续应用过程中，可能存在变动，后续会持续更新。
<br>
当温度超过 46 度时，风扇会按不同的转速进行散热，温度超过 86 度后会降低 NPU 的频率，算力也会下降，温度如果继续升高到 100 度，则会触发系统重启。
