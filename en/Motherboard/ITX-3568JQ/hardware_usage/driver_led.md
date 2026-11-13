# LED

## Introduction [step]

There are 2 LEDs on the iCore-3568JQ development board, as the following table shows:

| LED     | Pin name  | Pin number |
| ----    | ----      | ----       |
| Blue    | GPIO0_B6  | 14        |
| Yellow  | GPIO4_C4  | 148        |


Both LEDs can be controlled by using the LED device subsystem or by directly operating GPIO.

## Controlling LEDs by device [step]

Linux has its own LED subsystem for LED devices. In iCore-3568JQ, 2 LEDs are configured as LED class devices.

You can control them via `/sys/class/leds/`.

The default status of the two on-board LEDs are:

- Blue: Turn on after the system powers on.
- Yellow: Defined by the user.

You can change the behavior of each LED by using the `echo` command to write to its `brightness` property:

```bash
# echo 0 >/sys/class/leds/firefly:blue:power/brightness  //Blue led off
# echo 1 >/sys/class/leds/firefly:blue:power/brightness  //Blue led on
```

## Using trigger control LED [step]

Trigger contains a variety of ways to control the LED; the following two examples illustrate it.

- Simple trigger LED
- Complex trigger LED

For more information, please read the document `leds-class.txt`.

First of all, you need to know how many LEDs are to be defined and what the corresponding properties of the LEDs are.

Define LED node in file `kernel/arch/arm64/boot/dts/rockchip/rk3568j-firefly-itxq.dtsi`
```dts
firefly_leds: leds {
	status = "okay";
	compatible = "gpio-leds";
	power_led: power {
		label = "firefly:blue:power";
		linux,default-trigger = "ir-power-click";
		default-state = "on";
		gpios = <&gpio0 RK_PB6 GPIO_ACTIVE_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&led_power>;
	};
	user_led: user {
		label = "firefly:yellow:user";
		linux,default-trigger = "ir-user-click";
		default-state = "off";
		gpios = <&gpio4 RK_PC4 GPIO_ACTIVE_LOW>;
		pinctrl-names = "default";
		pinctrl-0 = <&led_user>;
	};
};
```


<Callout title="Note" type="warn">

The value of `compatible` must match the one in `drivers/leds/leds-gpio.c`.

</Callout>

### Simple trigger LED

As the name suggests, it is a simple trigger mode to control LEDs. The following configuration turns on the yellow LED by default, and the yellow LED of iCore-3568JQ will be lit after boot.

(1) Define the LED trigger; add the following in `kernel/drivers/leds/trigger/led-firefly-demo.c`:

```c
DEFINE_LED_TRIGGER(ledtrig_default_control);
```

(2) Register the trigger.

```c
led_trigger_register_simple("ir-user-click", &ledtrig_default_control);
```

(3) Control the LED.

```c
led_trigger_event(ledtrig_default_control, LED_FULL);     #yellow led on
```

(4) Enable LED demo.

`led-firefly-demo` is disabled by default. If needed, use the following patch to enable the demo driver:

```diff
--- a/kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
@@ -52,7 +52,7 @@
            led_demo: led_demo {
-                status = "disabled";
+                status = "okay";
                 compatible = "firefly,rk356x-led";
                 };
```

### Complex trigger LED

The following is a more complex example of controlling LEDs with the trigger mode: `timer trigger` makes the LED blink constantly.

The timer trigger needs to be configured in the kernel.

In the `kernel` directory, use `make menuconfig` and select the `timer trigger` driver as follows:

```text
Device Drivers
--->LED Support
   --->LED Trigger support
      --->LED Timer Trigger
```

Save the configuration and compile the kernel, then flash `kernel.img` to the iCore-3568JQ board. Use the serial port to enter the command, and the blue LED will blink at intervals.

```bash
echo "timer" > sys/class/leds/firefly\:blue\:power/trigger
```

The user can also use the `cat` command to get the available values for the trigger:

```console
root@rk3399_firefly_box:/ # cat sys/class/leds/firefly\:blue\:power/trigger
none rc-feedback test_ac-online test_battery-charging-or-full test_battery-charging
test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc1
ir-user-click [timer] heartbeat backlight default-on rfkill0 mmc2 rfkill1 rfkill2
```
