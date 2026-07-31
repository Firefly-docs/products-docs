# LED

## Introduction


| LED     | Pin name  | Pin number |
| ----    | ----      | ----       |
| power   | GPIO4_A5  | 133        |


LEDs can be controlled by using the LED device subsystem or by directly operating GPIO.

## Controlling LEDs by device

Linux has its own LED subsystem for LED devices. In ICORE-1126BQ38, LEDs are configured as LED class devices.You can control them via `/sys/class/leds/`.


You can change the behavior of each LED by using the echo command to write command to its brightness property:







Yellow light:

```
echo 1 > /sys/class/leds/\:power/brightness #Turn on the yellow light
echo 0 > /sys/class/leds/\:power/brightness #Turn off the yellow light
```



## Using trigger control LED

Trigger contains a variety of ways to control the LED, here with two examples to illustrate.

* Simple trigger LED
* Complex trigger LED

For more information, please read the document `leds-class.txt`.

First of all, we need to know how many LED definition, while the corresponding property of the LED is.

The board-level configuration is located in `sdk/kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-cam-1126bq38.dtsi`：

```
leds: leds {
        compatible = "gpio-leds";

        work {
                label = ":power";
                linux,default-trigger = "ir-power-click";
                gpios = <&gpio4 RK_PA5 GPIO_ACTIVE_HIGH>;
                pinctrl-names = "default";
                pinctrl-0 = <&led_power>;
                default-state = "on";
        };
};
```


Note: The value of `compatible` must match the one in `drivers/leds/leds-gpio.c`.

### Complex trigger LED

The following is the trigger mode control LED complex example, `timer trigger` is to let the LED to achieve constant light off effect.

We need to configure the timer trigger on the kernel.

In the `kernel` path using `make menuconfig`, in accordance with the following method to chose `timer trigger` driver.

```
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```
Save the configuration and compile the kernel, the `kernel.img` burn ICORE-1126BQ38 board. We can use the serial input command, you can see the blue light non-stop interval flashing.


```
# The yellow light is flashing
echo "timer" > /sys/class/leds/:power/trigger
```


The user can also use the `cat` command to get the available values for the trigger:


```
$ cat /sys/class/leds/:power/trigger
none ir-power-click rfkill-any rfkill-none test_ac-online test_battery-charging-or-full 
test_battery-charging test_battery-full test_battery-charging-blink-full-solid 
test_usb-online mmc0 [timer] heartbeat backlight default-on ir-user-click mmc1 
rfkill0 tcpm-source-psy-6-0022-online rfkill1 rfkill2
```


