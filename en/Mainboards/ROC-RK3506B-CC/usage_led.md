# LED

## Introduction


LEDs can be controlled by using the LED device subsystem or by directly operating GPIO.

## Controlling LEDs by device

Linux has its own LED subsystem for LED devices. In ROC-RK3506B-CC, LEDs are configured as LED class devices.You can control them via `/sys/class/leds/`.

The default status of the three on-board leds are:

* Blue: Turn on after the system powers on
* Red: defined by user

Users can control each LED by inputting commands to its `brightness` property through the `echo` command, for example:

* Blue LED
```
echo 0 >/sys/class/leds/\:power/brightness  //turn off the blue light
echo 1 >/sys/class/leds/\:power/brightness  //turn on the blue light
```

* Red LED
```
echo 0 >/sys/class/leds/\:user/brightness  //turn off the red light
echo 1 >/sys/class/leds/\:user/brightness  //turn on the red light
```



## Using trigger control LED

Trigger contains a variety of ways to control the LED, here with two examples to illustrate.

* Simple trigger LED
* Complex trigger LED

For more information, please read the document `leds-class.txt`.

First of all, we need to know how many LED definition, while the corresponding property of the LED is.

Defined LED node in file `kernel/arch/arm64/boot/dts/rockchip/rk3506b-firefly-roc-rk3506b-cc.dtsi`, like this:
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

Note: The value of `compatible` must match the one in `drivers/leds/leds-gpio.c`.

### Complex trigger LED

The following is the trigger mode control LED complex example, `timer trigger` is to let the LED to achieve constant on/off effect.

We need to configure the timer trigger on the kernel.

In the `kernel` path using `make menuconfig`, in accordance with the following method to chose `timer trigger` driver.

```
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```
LED Timer Trigger should be enabled by default. If not, enable it, save defconfig and re-build kernel to take effect.

Echo "timer" to trigger then we can see LED starts to blink.
```
echo "timer" > /sys/class/leds/\:user/trigger
```

The user can also use the `cat` command to get the available values for the trigger:

```
root@rk3506-buildroot:/# cat /sys/class/leds/\:user/trigger
none rfkill-any rfkill-none kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock kbd-ctrlrlock mmc0 [timer] rfkill0
```
