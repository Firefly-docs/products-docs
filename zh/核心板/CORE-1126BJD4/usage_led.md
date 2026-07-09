# LED 使用

## 前言


| LED     | Pin name  | Pin number |
| ----    | ----      | ----       |
| Up      | GPIO7_A4  | 228        |
| Down    | GPIO7_A6  | 230        |


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 AIO-1126BJD4开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED：






上层灯：

```
echo 1 > /sys/class/leds/\:power/brightness #上层灯亮
echo 0 > /sys/class/leds/\:power/brightness #上层灯灭
```

下层灯：

```
echo 1 > /sys/class/leds/\:user/brightness #下层灯亮
echo 0 > /sys/class/leds/\:user/brightness #下层灯灭
```



## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。

板级的配置在`sdk/kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126b-firefly-aio-1126bjd4.dtsi`：

```
leds: leds {
    compatible = "gpio-leds";

    work {
        label = ":power";
        linux,default-trigger = "ir-power-click";
        gpios = <&gpio7 RK_PA4 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_power>;
        default-state = "on";
    };

    user {
        label = ":user";
        linux,default-trigger = "ir-user-click";
        gpios = <&gpio7 RK_PA6 GPIO_ACTIVE_HIGH>;
        pinctrl-names = "default";
        pinctrl-0 = <&led_user>;
        default-state = "off";
    };
};
```


### Complex trigger LED

如下是 trigger 方式控制 LED 复杂一点的例子，`timer trigger` 就是让 LED 达到不断亮灭的效果：

我们需要在内核把 timer trigger 配置上。

在 `kernel` 路径下使用 `make menuconfig`，按照如下方法将 timer trigger 驱动选中。

```
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```
保存配置并编译内核，把 `boot.img` 烧到 AIO-1126BJD4板子上 我们可以使用串口输入命令，就可以看到蓝灯不停的间隔闪烁。


```
echo "timer" > /sys/class/leds/:user/trigger
```


用户还可以使用 `cat` 命令获取 trigger 的可用值：

```
$ cat /sys/class/leds/:user/trigger
none ir-power-click rfkill-any rfkill-none test_ac-online test_battery-charging-or-full 
test_battery-charging test_battery-full test_battery-charging-blink-full-solid 
test_usb-online mmc0 [timer] heartbeat backlight default-on ir-user-click mmc1 
rfkill0 tcpm-source-psy-6-0022-online rfkill1 rfkill2
```

