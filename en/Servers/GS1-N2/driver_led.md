# LED

There are 2 LEDs on the GS1-N2 board:

| Color | Location | Symbol | Description |
| ---- | :--: | ---- | ---- |
| Yellow | Up | DIY_LED | User controls |
| Green | Down | WORK_LED | Power on |

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

