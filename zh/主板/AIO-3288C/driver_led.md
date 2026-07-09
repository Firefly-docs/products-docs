# LED 使用

## 前言

AIO-3288C 开发板上有 2 个 LED 灯，如下表所示：


```c
LED         GPIO ref.       GPIO number
Blue        GPIO8_A1        257
Red         GPIO8_A6        262
```


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 AIO-3288C 开发板中的两个 LED 均以设备的形式被定义。

用户可以通过 `/sys/class/leds/` 目录控制这两个 LED。  

更详细的说明请参考 [leds-class.txt](https://www.kernel.org/doc/Documentation/leds/leds-class.txt) 。

开发板上的 LED 的默认状态为：


* Red: 系统上电时打开
* Blue：用户自定义


用户可以通过 echo 向其 trigger 属性输入命令控制每一个 LED：

```bash
root@firefly:~ # echo none >/sys/class/leds/firefly:blue:power/trigger
root@firefly:~ # echo default-on >/sys/class/leds/firefly:blue:power/trigger
```

用户还可以使用 cat 命令获取 trigger 的可用值：

```bash
root@firefly:~ # cat /sys/class/leds/firefly:blue:power/trigger
none [ir-power-click] test_ac-online test_battery-charging-or-full test_battery-charging
test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc1 mmc2
backlight default-on rfkill0 rfkill1 rfkill2
```

## 在内核中操作 LED

在内核中操作 LED 的步骤如下：

1、在 dts 文件中定义 LED 节点“leds”
在 `kernel/arch/arm/boot/dts/firefly-rk3288-aio-3288c.dts` 文件中定义LED节点，具体定义如下：

```dts
leds {
    compatible = "gpio-leds";
    power {
        label = "firefly:blue:power";
        linux,default-trigger = "ir-power-click";
        default-state = "on";
        gpios = <&gpio8 GPIO_A1 GPIO_ACTIVE_HIGH>;
    };

    user {
        label = "firefly:yellow:user";
        linux,default-trigger = "ir-user-click";
        default-state = "off";
        gpios = <&gpio8 GPIO_A2 GPIO_ACTIVE_HIGH>;
    };
};
```

注意：compatible 的值要跟 `drivers/leds/leds-gpio.c` 中的 `.compatible` 的值要保持一致。

2、在驱动文件包含头文件

```c
#include <linux/leds.h>
```

3、在驱动文件中控制 LED。

（1）、定义 LED 触发器

```c
DEFINE_LED_TRIGGER(ledtrig_ir_click);
```

（2）、注册该触发器

```c
led_trigger_register_simple("ir-power-click", &ledtrig_ir_click);
```

（3）、控制 LED 的亮灭。

```c
led_trigger_event(ledtrig_ir_click, LED_FULL); //亮
led_trigger_event(ledtrig_ir_click, LED_OFF); //灭
```