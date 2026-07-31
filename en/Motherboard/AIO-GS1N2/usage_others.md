# Others
## BUZZER

|ON|OFF|
|---|---|
|echo 1 > /sys/class/leds/beeper_led/brightness|echo 0 > /sys/class/leds/beeper_led/brightness|

## LED 

||ON|OFF|
|---|---|---|
|POWER|echo 1 > /sys/class/leds/ext_pwr_led1/brightness|echo 0 > /sys/class/leds/ext_pwr_led1/brightness|
|STORAGE|echo 1 > /sys/class/leds/ext_sata_led2/brightness|echo 0 > /sys/class/leds/ext_sata_led2/brightness|
|NETWORK|echo 1 > /sys/class/leds/ext_net_led3/brightness|echo 0 > /sys/class/leds/ext_net_led3/brightness|


## Power 
The main module provides **power supply** to the sub module, **startup** operation.

||ON|OFF|
|---|---|---|
|power supply|echo 1 > /sys/class/leds/res_core_en/brightness|echo 0 > /sys/class/leds/res_core_en/brightness|
|startup|echo 1 > /sys/class/leds/res_power/brightness  |echo 0 > /sys/class/leds/res_power/brightness|