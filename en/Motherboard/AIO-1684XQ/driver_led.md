# LED

There are 2 LEDs on the AIO-1684XQ board:
| Color | Location | Symbol | Description |
| ---- | :--: | ---- | ---- |
| Yellow | Right | DIY_LED | User controls |
| Green | Left (near the power inlet) | WORK_LED | Power on |
Example command to control LED light state (requires root privileges):

````
# yellow light on
echo 1 >/sys/class/leds/DIY_LED/brightness
# yellow light off
echo 0 >/sys/class/leds/DIY_LED/brightness

# green light on
echo 1 >/sys/class/leds/WORK_LED/brightness
# green light off
echo 0 >/sys/class/leds/WORK_LED/brightness
````

