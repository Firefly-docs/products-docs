# LED 使用

## 前言

| LED     | Pin name  | Pin number |
| ----    | ----      | ----       |
| Up      | GPIO0_C0  | 16        | 
| Down    | GPIO0_A4  | 4        |

可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 Core-1126-JD4开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


*   Up:  用户自定义状态
*   Down：系统上电时打开状态


用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED：



上层灯：
```
echo 1 > /sys/class/leds/firefly\:yellow\:user/brightness #上层灯亮
echo 0 > /sys/class/leds/firefly\:yellow\:user/brightness #上层灯灭
```

下层灯：
```
echo 1 > /sys/class/leds/firefly\:blue\:power/brightness #下层灯亮
echo 0 > /sys/class/leds/firefly\:blue\:power/brightness #下层灯灭
```



## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。

板级的配置在`sdk/kernel/arch/arm/boot/dts/rv1126-firefly-rk809.dtsi`：
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

# pinctrl 需要添加 GPIO 复用控制
leds {
	led_power: led-power {
            rockchip,pins = <0 RK_PA4 RK_FUNC_GPIO &pcfg_pull_none>;
        };

        led_user: led-user {
            rockchip,pins = <0 RK_PC0 RK_FUNC_GPIO &pcfg_pull_none>;
        };
};
```



注意：`compatible` 的值要跟 `drivers/leds/leds-pwm.c` 中的 `.compatible` 的值要保持一致。

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
保存配置并编译内核，把 `zboot.img` 烧到 Core-1126-JD4板子上 我们可以使用串口输入命令，就可以看到对应的灯不停的间隔闪烁。

```
# 上层灯闪烁
echo "timer" > /sys/class/leds/firefly\:yellow\:user/trigger

# 下层灯闪烁
echo "timer" > /sys/class/leds/firefly\:blue\:power/trigger
```

用户还可以使用 `cat` 命令获取 trigger 的可用值：


```
$ cat /sys/class/leds/firefly\:yellow\:user/trigger
none rfkill-any rfkill-none mmc0 mmc2 mmc1 [timer] oneshot heartbeat default-on rfkill0 rfkill1 rfkill2 rfkill3
```

