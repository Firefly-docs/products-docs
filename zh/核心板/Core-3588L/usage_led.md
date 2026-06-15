# LED 使用

## 前言


可通过使用 LED 设备子系统或者直接操作 GPIO 控制该 LED。

## 以设备的方式控制 LED

标准的 Linux 专门为 LED 设备定义了 LED 子系统。 在 AIO-3588L开发板中的LED 均以设备的形式被定义。用户可以通过 `/sys/class/leds/` 目录控制LED。

开发板上的 LED 的默认状态为：


用户可以通过 `echo` 命令向其 `brightness` 属性输入命令控制每一个 LED：
```
echo 0 >/sys/class/leds/:user/brightness  //灯灭
echo 1 >/sys/class/leds/:user/brightness  //灯亮
```

## 使用 trigger 方式控制 LED

Trigger 包含多种方式可以控制 LED，这里就用两个例子来说明。

*    Simple trigger LED
*    Complex trigger LED

更详细的说明请参考 `leds-class.txt` 。

首先我们需要知道定义多少个 LED，同时对应的 LED 的属性是什么。



注意：`compatible` 的值要跟 `drivers/leds/leds-gpio.c` 中的 `.compatible` 的值要保持一致。

### Simple trigger LED

按名字来是看就是简单的触发方式控制 LED，如下就默认打开黄灯，AIO-3588L开机后黄灯就亮。

（1）定义 LED 触发器在 `kernel-5.10/drivers/leds/trigger/led-firefly-demo.c` 文件中有如下添加：

```
DEFINE_LED_TRIGGER(ledtrig_default_control);
```

（2）注册该触发器

```
led_trigger_register_simple("ir-user-click", &ledtrig_default_control);
```

（3）控制 LED 的亮。

```
led_trigger_event(ledtrig_default_control, LED_FULL);     #led on
```

（4）打开LED demo

led-firefly-demo 默认没有打开，如果需要的话可以使用以下补丁打开 demo 驱动：

```
--- a/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-demo.dtsi
+++ b/kernel-5.10/arch/arm64/boot/dts/rockchip/rk3588-firefly-demo.dtsi
@@ -52,7 +52,7 @@
led_demo: led_demo {
-     status = "disabled";
+     status = "okay";
      compatible = "firefly,rk3588-led";
};
```

### Complex trigger LED

如下是 trigger 方式控制 LED 复杂一点的例子，`timer trigger` 就是让 LED 达到不断亮灭的效果：

我们需要在内核把 timer trigger 配置上。

在 `kernel-5.10` 路径下使用 `make menuconfig`，按照如下方法将 timer trigger 驱动选中。

```
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```
保存配置并编译内核，把 `kernel.img` 烧到 AIO-3588L板子上 我们可以使用串口输入命令，就可以看到蓝灯不停的间隔闪烁。

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
