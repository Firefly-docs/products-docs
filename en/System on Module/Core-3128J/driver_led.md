# LED Use

## Introduction
There are 2 LEDs on the Firefly-RK3128 development board, as the following table shows:  
![](../../../rk3128_img/Core-3128J/driver_table2.png)

Both are programmable using ether LEDs class devices or GPIOs.

## LEDs as Class Devices

Linux has its own LED subsystem for LED devices. In Firefly-RK3128, 2 LEDs are configured as LED class devices.  
You can control them via `/sys/class/leds/`.  
For more information, please read the document [leds-class.txt](https://www.kernel.org/doc/Documentation/leds/leds-class.txt) .    
The default status of the two on-board leds are:

* Blue: on after the system powers on.
* Yellow: defined by user.

You can change the behavior of each LED by using the echo command to write command to its trigger property:  
```
root@firefly:~ # echo none >/sys/class/leds/firefly:blue:power/trigger
root@firefly:~ # echo default-on >/sys/class/leds/firefly:blue:power/trigger
```

You can also use the cat command to list all available values of the trigger attribute:
```
root@firefly:~ # cat /sys/class/leds/firefly:blue:power/trigger
none [ir-power-click] test_ac-online test_battery-charging-or-full test_battery-charging
test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc1 mmc2
backlight default-on rfkill0 rfkill1 rfkill2
```

## Controlling LED in the Kernel

The steps to operate LED in kernel are as follows:

1. Define LED node in the dts file.  
Define LED node in file kernel/arch/arm/boot/dts/rk3128-fireprime.dts:
```
leds {
		compatible = "gpio-leds";
    	power {
    		label = "firefly:blue:power";
        	linux,default-trigger = "ir-power-click";
        	default-state = "on";
        	gpios = <&gpio1 GPIO_C7 GPIO_ACTIVE_LOW>;
    	};
    
    	user {
    		label = "firefly:yellow:user";
            linux,default-trigger = "ir-user-click";
            default-state = "off";
            gpios-v01 = <&gpio2 GPIO_A3 GPIO_ACTIVE_LOW>;
        };
};
```
Note: The value of .compatible must match the one in drivers/leds/leds-gpio.c.

2. Include head files in the driver.
```
#include <linux/leds.h>
```

3. Control LED in driver file.

(1). Define a trigger.
```
DEFINE_LED_TRIGGER(ledtrig_ir_click);
```
(2). Register this trigger.
```
led_trigger_register_simple("ir-power-click", &ledtrig_ir_click);
```
(3). Control on/off of LED.
```
led_trigger_event(ledtrig_ir_click, LED_FULL); //ON
led_trigger_event(ledtrig_ir_click, LED_OFF); //OFF
```