# Others
## BUZZER

|ON|OFF|
|---|---|
|echo 1 > /sys/class/leds/:beeper_led/brightness|echo 0 > /sys/class/leds/:beeper_led/brightness|

## LED 

||ON|OFF|
|---|---|---|
|POWER|echo 1 > /sys/class/leds/:work/brightness|echo 0 > /sys/class/leds/:work/brightness|
|STORAGE|echo 1 > /sys/class/leds/:sata_led/brightness|echo 0 > /sys/class/leds/:sata_led/brightness|
|NETWORK|echo 1 > /sys/class/leds/:user/brightness|echo 0 > /sys/class/leds/:user/brightness|