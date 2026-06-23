# LED Usage

There are 2 LED lights on the AIO-186JD4 development board ([As shown in the figure](interface_definition.html#zheng-ji-jie-kou-ding-yi) ）:

| Color  | Position | Label     | Description  |
| ------ | :------: | --------- | ------------ |
| Yellow |    Up    | DIY\_LED  | User Control |
| Green  |   Down   | WORK\_LED | Power On     |

Example command to control the LED light status (requires root privileges):

```
# 黄灯亮
echo 1 >/sys/class/leds/DIY_LED/brightness
# 黄灯灭
echo 0 >/sys/class/leds/DIY_LED/brightness

# 绿灯亮
echo 1 >/sys/class/leds/WORK_LED/brightness
# 绿灯灭
echo 0 >/sys/class/leds/WORK_LED/brightness
```