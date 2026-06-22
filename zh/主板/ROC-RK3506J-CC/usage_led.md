# LED 使用

## 前言

ROC-RK3506J-CC开发板上有两个LED灯，由扩展 gpio PCA9555 控制，如下表所示：

| LED     | Pin name  | Pin number | 
| ----    | ----      | ----       | 
| Blue    | PCA_IO1_6 | 286        |
| Red     | PCA_IO0_4 | 276        |


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 ROC-RK3506J-CC开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


* Blue: 系统上电时打开状态
* Red: 用户自定义状态

用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED，比如：

* Blue LED
```
echo 0 >/sys/class/leds/\:power/brightness  //蓝灯灭
echo 1 >/sys/class/leds/\:power/brightness  //蓝灯亮
```

* Red LED
```
echo 0 >/sys/class/leds/\:user/brightness  //红灯灭
echo 1 >/sys/class/leds/\:user/brightness  //红灯亮
```



## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。

在 `kernel/arch/arm/boot/dts/rk3506b-firefly-roc-rk3506b-cc.dtsi` 文件中定义 LED 节点，可以用 pwm 控制也可以用 gpio 控制，类似这样：
```
#if LED_GPIO_OR_PWM
	firefly_leds: leds {
		compatible = "pwm-leds";
		status = "okay";
		power_led {
			label = ":power";
			pwms = <&pwm2_8ch_0 0 50000 0>;
			max-brightness = <255>;
			default-state = "on";
			linux,default-trigger = "ir_led";
		};
		user_led {
			label = ":user";
			pwms = <&pwm2_8ch_2 0 50000 0>;
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
			label = ":power";
			linux,default-trigger = "ir-power-click";
			default-state = "on";
			gpios = <&pca9555 PCA_IO1_6 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
		};
		user_led: user {
			label = ":user";
			linux,default-trigger = "ir-user-click";
			default-state = "off";
			gpios = <&pca9555 PCA_IO0_4 GPIO_ACTIVE_HIGH>;
			pinctrl-names = "default";
		};
	};
#endif
```

注意：`compatible` 的值要跟 `drivers/leds/leds-gpio.c` 中的 `.compatible` 的值要保持一致。

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
echo "timer" > /sys/class/leds/\:user/trigger
```

用户还可以使用 `cat` 命令获取 trigger 的可用值：

```
root@rk3506-buildroot:/# cat /sys/class/leds/\:user/trigger
none rfkill-any rfkill-none kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock kbd-ctrlrlock mmc0 [timer] rfkill0
```
