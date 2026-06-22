# LED

## Introduction

| LED     | Pin name  | Pin number |
| ----    | ----      | ----       |
| Up      | GPIO0_C0  | 16        | 
| Down    | GPIO0_A4  | 4        |

LEDs can be controlled by using the LED device subsystem or by directly operating GPIO.

## Controlling LEDs by device

Linux has its own LED subsystem for LED devices. In C40PL, LEDs are configured as LED class devices.You can control them via `/sys/class/leds/`.


*   Up: User-defined status
*   Down: The system is turned on when it is powered on.


You can change the behavior of each LED by using the echo command to write command to its brightness property:




Upper Lights:
```
echo 1 > /sys/class/leds/firefly\:yellow\:user/brightness # Upper level lights on
echo 0 > /sys/class/leds/firefly\:yellow\:user/brightness # Upper lights off
```

Lower level lights:
```
echo 1 > /sys/class/leds/firefly\:blue\:power/brightness # Lower level lights on
echo 0 > /sys/class/leds/firefly\:blue\:power/brightness # Lower level lights off
```




## Using trigger control LED

Trigger contains a variety of ways to control the LED, here with two examples to illustrate.

* Simple trigger LED
* Complex trigger LED

For more information, please read the document `leds-class.txt`.

First of all, we need to know how many LED definition, while the corresponding property of the LED is.

Board:`sdk/kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi`：
```
leds {
    compatible = "gpio-leds";

	work {
        label = "firefly:blue:power";
        linux,default-trigger = "ir-power-click";
        gpios = <&gpio0 RK_PA4 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_power>;
        default-state = "on";
	};
    user {
        label = "firefly:yellow:user";
        linux,default-trigger = "ir-user-click";
        gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_user>;
        default-state = "on";
	};
};

# pinctrl needs to add GPIO multiplexing control
leds {
	led_power: led-power {
            rockchip,pins = <0 RK_PA4 RK_FUNC_GPIO &pcfg_pull_none>;
        };

        led_user: led-user {
            rockchip,pins = <0 RK_PC0 RK_FUNC_GPIO &pcfg_pull_none>;
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
Save the configuration and compile the kernel, and burn `zboot.img` to the C40PL board. We can use the serial port to input commands, and we can see the corresponding lights flashing at intervals.

```
# Upper lights flashing
echo "timer" > /sys/class/leds/firefly\:yellow\:user/trigger

# Lower level lights flashing
echo "timer" > /sys/class/leds/firefly\:blue\:power/trigger
```


The user can also use the `cat` command to get the available values for the trigger:

```
$ cat /sys/class/leds/firefly\:yellow\:user/trigger
none rfkill-any rfkill-none mmc0 mmc2 mmc1 [timer] oneshot heartbeat default-on rfkill0 rfkill1 rfkill2 rfkill3
```

