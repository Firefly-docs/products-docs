# LED Use

## Foreword

AIO-3288J development board has 2 LEDs, as shown in the table below:


```c
LED         GPIO ref.       GPIO number
Blue        GPIO8_A1        257
Red         GPIO8_A6        262
```


The user can use LED equipment subsystem or directly operate GPIO to control this LED.

## Control LED in the form of equipment

Standard Linux specially defines LED subsystem for LED equipment. Two LEDs on AIO-3288C development board are defined in the form of equipment.

The user can control these two LEDs through `/sys/class/leds/` catalog.

For more detailed descriptions, please refer to [leds-class.txt](https://www.kernel.org/doc/Documentation/leds/leds-class.txt).

The default states of LED on development board include:


* Red: turn on when the system is electrified
* Blue: user defined


The user can input command to trigger attribute through echo to control each LED:

```bash
root@firefly:~ # echo none >/sys/class/leds/firefly:blue:power/trigger
root@firefly:~ # echo default-on >/sys/class/leds/firefly:blue:power/trigger
```

The user can use cat command to obtain the available value of trigger:

```bash
root@firefly:~ # cat /sys/class/leds/firefly:blue:power/trigger
none [ir-power-click] test_ac-online test_battery-charging-or-full test_battery-charging
test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc1 mmc2
backlight default-on rfkill0 rfkill1 rfkill2
```

## Operate LED in kernel

The steps to operate LED in kernel are as follows:

1.Define LED node “leds” in dts file
Define LED node in `kernel/arch/arm/boot/dts/firefly-rk3288-aio-3288c.dts` file; the specific definition is as follows:

```dts
 leds {
        compatible = "gpio-leds";
        power {
            label = "firefly:blue:power";
            linux,default-trigger = "ir-power-click";
            default-state = "on";
            gpios = <&gpio8 GPIO_A1 GPIO_ACTIVE_HIGH>;
        };
        user{
            label = "firefly:yellow:user";
            linux,default-trigger = "ir-user-click";
            default-state = "off";
            gpios = <&gpio8 GPIO_A2 GPIO_ACTIVE_HIGH>;
        };
  };
```

Note: the value of compatible shall coincide to the value of `.compatible` in `drivers/leds/leds-gpio.c`.

2.Include header file into driver file

```c
#include <linux/leds.h>
```

3.Control LED in driver file.

(1). Define LED trigger

```c
DEFINE_LED_TRIGGER(ledtrig_ir_click);
```

(2). Register this trigger

```c
led_trigger_register_simple("ir-power-click", &ledtrig_ir_click);
```

(3). Control on/off of LED.

```c
led_trigger_event(ledtrig_ir_click, LED_FULL); //on
led_trigger_event(ledtrig_ir_click, LED_OFF); //off
```