# LED 使用

## 前言


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 CQ38W-3576开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED，比如：

```
echo 0 >/sys/class/leds/:user/brightness  //灯灭
echo 255 >/sys/class/leds/:user/brightness  //灯亮
```

## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。

在 `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-ext-icore-3576q38.dtsi` 文件中定义 LED 节点，可以用 pwm 控制也可以用 gpio 控制，类似这样：
```
#if LED_GPIO_OR_PWM
	firefly_leds: leds {
		compatible = "pwm-leds";
		status = "okay";
		power_led {
			label = ":power";
			pwms = <&pwm2_8ch_0 0 50000 0>; //blue
			max-brightness = <255>;
			default-state = "on";
			linux,default-trigger = "ir_led";
		};
		user_led {
			label = ":user";
			pwms = <&pwm2_8ch_2 0 50000 0>; //red
			max-brightness = <255>;
			default-state = "off";
			linux,default-trigger = "none";
		};
		diy_led {
			label = ":diy";
			pwms = <&pwm2_8ch_1 0 50000 0>; //green
			max-brightness = <255>;
			default-state = "off";
			linux,default-trigger = "none";
		};
	};
#else
	firefly_leds: leds {
		compatible = "gpio-leds";
		status = "okay";
		power_led: power {
			label = ":power"; //blue led
			linux,default-trigger = "ir-power-click";
			default-state = "on";
			gpios = <&gpio2 RK_PD0 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
			pinctrl-0 = <&led_power>;
		};
		user_led: user {
			label = ":user"; //red led
			linux,default-trigger = "ir-user-click";
			default-state = "off";
			gpios = <&gpio2 RK_PD2 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
			pinctrl-0 = <&led_user>;
		};
		diy_led: diy {
			label = ":diy"; //green led
			linux,default-trigger = "ir-user-click";
			default-state = "off";
			gpios = <&gpio2 RK_PD1 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
			pinctrl-0 = <&led_diy>;
		};
	};
#endif
```

注意：`compatible` 的值要跟 `drivers/leds/leds-gpio.c` 中的 `.compatible` 的值要保持一致。

### Simple trigger LED

按名字来是看就是简单的触发方式控制 LED，请参考 `kernel/drivers/leds/trigger/led-firefly-demo.c`

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
默认应该就是开启的，如果没有，打开后要保存配置并编译烧录内核。

之后我们可以使用串口输入命令，就可以看到灯不停的间隔闪烁。

```
echo "timer" > /sys/class/leds/:user/trigger
```

用户还可以使用 `cat` 命令获取 trigger 的可用值：

```
# cat /sys/class/leds/:user/trigger
none ir-power-click rfkill-any rfkill-none test_ac-online test_battery-charging-or-full 
test_battery-charging test_battery-full test_battery-charging-blink-full-solid 
test_usb-online mmc0 [timer] heartbeat backlight default-on ir-user-click mmc1 
rfkill0 tcpm-source-psy-6-0022-online rfkill1 rfkill2
```
