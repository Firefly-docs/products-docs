# LED 使用

## 前言


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 CAM-C1126S2U开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


*   IR:  IR 红外补光灯状态
*   RGB：白色 LED 补光灯状态


用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED：




IR 红外补光灯：
```
echo 77 > /sys/class/leds/PWM-IR/brightness #红外补光灯亮
echo 0 > /sys/class/leds/PWM-IR/brightness  #红外补光灯灭
```

白色 LED 补光灯：
```
echo 15 > /sys/class/leds/PWM-RGB/brightness # 白色 LED 补光灯亮
echo 0 >  /sys/class/leds/PWM-RGB/brightness # 白色 LED 补光灯灭
```

说明：

由于 CAM-C1126S2U 的 IR 红外补光灯和白色 LED 补光灯是由 PWM 驱动的。所以可以调整 PWM 占空比来调整灯光的亮度。**echo 输入 0 ~ 255 区间范围的值来控制灯光的亮度。**

**注意：**

1. **IR 红外补光灯是功率较大的部件。它是该设备主要功耗来源。由于 CAM-C1126S2U 设备比较紧凑，设置过高的 IR 红外补光灯亮度会加大设备功耗，导致设备发热严重，设备温度过高会导致性能下降。**
2. **由于 CAM-C1126S2U 只有一个 TYPE-C 供电口。设置过高的 IR 红外补光灯亮度会导致电流的供电需求加大，如果是用笔记本或者 PC 前置的 USB 接口来供电，很容易出现供电电流不足的情况。在这种供电电流不足情况下，设备可能会一直重启或者断开与上位机的连接。**

不同应用场景的解决办法：
1. 如果是 Facial_Gate 的应用场景，不需要通过 usb 与上位机进行数据通信的情况下。可以使用较大功率的电源给设备供电。如使用 5V 2A 或者 5V 3A 的电源适配器给设备供电。
2. 如果是需要通过 usb 与上位机进行数据通信的情况，则使用可以外接电源的 usb hub 来给 CAM-C1126S2U 设备电源供电。


## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。

板级的配置在`sdk/kernel/arch/arm/boot/dts/rv1109-firefly-ai-cam.dtsi`：
```
pwmleds {
	compatible = "pwm-leds";

	pwmi_ir {
		label = "PWM-IR";
		pwms = <&pwm7 0 25000 0>;
		max-brightness = <255>;
	};

	pwmi_rgb {
		label = "PWM-RGB";
		pwms = <&pwm2 0 25000 0>;
		max-brightness = <255>;
		init-brightness = <15>;
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
保存配置并编译内核，把 `kernel.img` 烧到 CAM-C1126S2U板子上 我们可以使用串口输入命令，就可以看到蓝灯不停的间隔闪烁。


```
# IR 红外补光灯闪烁
echo "timer" > /sys/class/leds/PWM-IR/trigger

# 白色 LED 补光灯闪烁
echo "timer" > /sys/class/leds/PWM-RGB/trigger
```
用户还可以使用 `cat` 命令获取 trigger 的可用值：


```
$ cat /sys/class/leds/PWM-RGB/trigger
none rfkill-any rfkill-none mmc0 [timer] oneshot heartbeat default-on
```

