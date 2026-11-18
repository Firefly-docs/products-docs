# LED Usage

## Introduction [step]

Firefly-R3572A has 3 on-board LEDs driven directly by GPIO (DTS node `firefly-leds`, `compatible = "gpio-leds"`):

| LED | DTS label | GPIO | Default trigger | Power-on state |
| --- | --- | --- | --- | --- |
| Power LED | `:power` | GPIO0_PC0 | `ir-power-click` | on |
| User LED | `:user` | GPIO0_PC1 | `ir-user-click` | off |
| DIY LED | `:diy` | GPIO3_PD6 | `none` | off |

They can be controlled through the LED subsystem (sysfs) or by operating the GPIO directly. The node names, GPIOs and pinctrl above are taken from the board-level DTS `rk3572-firefly-r3572a.dtsi` of the Firefly-R3572A Linux SDK (enabled/disabled by the `N_LED` switch in the board dts); refer to your SDK for the actual code.

## Controlling LEDs as Devices [step]

The Linux LED subsystem creates a directory for each LED under `/sys/class/leds/`. Control the brightness through the `brightness` attribute:

```bash
# List the on-board LEDs
ls /sys/class/leds/

# Turn the DIY LED on / off (1 on, 0 off)
echo 1 > /sys/class/leds/:diy/brightness
echo 0 > /sys/class/leds/:diy/brightness
```

Show the available (or currently used) triggers:

```bash
cat /sys/class/leds/:diy/trigger
```

## Controlling LEDs with Triggers [step]

LED triggers can realize automatic blinking, event-driven lighting and other effects. For example, to make the DIY LED blink periodically with a timer:

```bash
# Make sure the LED Timer Trigger is enabled in the kernel
# (Device Drivers > LED Support > LED Trigger support > LED Timer Trigger)
echo timer > /sys/class/leds/:diy/trigger
```

Other triggers work in the same way, for example a heartbeat:

```bash
echo heartbeat > /sys/class/leds/:diy/trigger
```

Cancel the trigger and restore manual control:

```bash
echo none > /sys/class/leds/:diy/trigger
```

## DTS Configuration Reference [step]

The LED node definition of Firefly-R3572A (in `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a.dtsi`):

```text
firefly_leds: firefly-leds {
    compatible = "gpio-leds";
    status = "okay";
    power_led: power {
        label = ":power";
        linux,default-trigger = "ir-power-click";
        gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
        default-state = "on";
    };
    user_led: user {
        label = ":user";
        linux,default-trigger = "ir-user-click";
        gpios = <&gpio0 RK_PC1 GPIO_ACTIVE_HIGH>;
        default-state = "off";
    };
    diy_led: diy {
        label = ":diy";
        linux,default-trigger = "none";
        gpios = <&gpio3 RK_PD6 GPIO_ACTIVE_HIGH>;
        default-state = "off";
    };
};
```

<Callout title="Note" type="info">

`compatible = "gpio-leds"` must be consistent with the `.compatible` in the driver `kernel-6.12/drivers/leds/leds-gpio.c`. To change the default trigger or add an LED, modify the board-level DTS and rebuild/reflash the firmware accordingly.

</Callout>
