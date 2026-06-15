# LED

## Introduction


LEDs can be controlled by using the LED device subsystem or by directly operating GPIO.

## Controlling LEDs by device

Linux has its own LED subsystem for LED devices. In CQ38W-3576, LEDs are configured as LED class devices.You can control them via `/sys/class/leds/`.

The default status of the three on-board leds are:

You can change the behavior of each LED by using the echo command to write command to its brightness property:

```
echo 0 >/sys/class/leds/:user/brightness  // led off
echo 255 >/sys/class/leds/:user/brightness  // led on
```

## Using trigger control LED

Trigger contains a variety of ways to control the LED, here with two examples to illustrate.

* Simple trigger LED
* Complex trigger LED

For more information, please read the document `leds-class.txt`.

First of all, we need to know how many LED definition, while the corresponding property of the LED is.

Defined LED node in file `kernel/arch/arm64/boot/dts/rockchip/rk3576-firefly-ext-icore-3576q38.dtsi`, like this:
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

Note: The value of `compatible` must match the one in `drivers/leds/leds-gpio.c`.

### Simple trigger LED

It is a simple trigger mode to control LEDs, please refer to `kernel/drivers/leds/trigger/led-firefly-demo.c`

### Complex trigger LED

The following is the trigger mode control LED complex example, `timer trigger` is to let the LED to achieve constant on/off effect.

We need to configure the timer trigger on the kernel.

In the `kernel-5.10` path using `make menuconfig`, in accordance with the following method to chose `timer trigger` driver.

```
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```
LED Timer Trigger should be enabled by default. If not, enable it, save defconfig and re-build kernel to take effect.

Echo "timer" to trigger then we can see LED starts to blink.
```
echo "timer" > /sys/class/leds/:user/trigger
```

The user can also use the `cat` command to get the available values for the trigger:

```
# cat /sys/class/leds/:user/trigger
none ir-power-click rfkill-any rfkill-none test_ac-online test_battery-charging-or-full 
test_battery-charging test_battery-full test_battery-charging-blink-full-solid 
test_usb-online mmc0 [timer] heartbeat backlight default-on ir-user-click mmc1 
rfkill0 tcpm-source-psy-6-0022-online rfkill1 rfkill2
```
