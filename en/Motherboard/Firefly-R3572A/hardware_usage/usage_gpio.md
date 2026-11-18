# GPIO Usage

## Introduction [step]

GPIO, short for General-Purpose Input/Output, is a generic pin that can be dynamically configured and controlled by software at runtime. All GPIOs default to input mode after power-on, can be pulled up or down by software, and can also be used as interrupt pins with programmable drive strength.

Firefly-R3572A breaks out a GPIO signal (`GPIO1_C7_d`, pull-up 10K) on the J1 dual-row pin header, which can be configured as input or output. On Linux, the libgpiod tool is recommended.

## Hardware Connection [step]

| Header Pin | Signal | Level |
| --- | --- | --- |
| 20 | GPIO1_C7_d | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="Note" type="warn">

The GPIO level is 1.8 V. Do not connect 3.3 V or 5 V signals directly, otherwise the chip may be damaged.

</Callout>

## GPIO Number Calculation [step]

The GPIO number of RK3572 is calculated as `bank × 32 + group × 8 + index`:

* Bank: the `1` in `GPIO1_C7_d`
* Group letter: A=0, B=1, C=2, D=3, so `C` is 2
* Index: `7`

Therefore the system number of `GPIO1_C7` is:

```text
1 × 32 + 2 × 8 + 7 = 55
```

The device tree property description of `GPIO1_C7` is `<&gpio1 55 GPIO_ACTIVE_HIGH>`. According to the macro definitions in `kernel-6.12/include/dt-bindings/pinctrl/rockchip.h`, it can also be written as `<&gpio1 RK_PC7 GPIO_ACTIVE_HIGH>`.

When GPIO1_C7 is not multiplexed by other peripherals, it can also be exported through the sysfs interface:

```shell
:/ # echo 55 > /sys/class/gpio/export
:/ # ls /sys/class/gpio/gpio55
active_low  device  direction  edge  power  subsystem  uevent  value
:/ # cat /sys/class/gpio/gpio55/direction
in
```

## Output Control [step]

Configure the GPIO as output and set the level (libgpiod):

```bash
gpioset gpiochip1 7=1    # output high
gpioset gpiochip1 7=0    # output low
```

## Input Reading [step]

Configure the GPIO as input and read the level:

```bash
gpioget gpiochip1 7
```

## Kernel Driver [step]

Add the driver resource description in the DTS file:

```text
kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-demo.dtsi

/{
    gpio_demo: gpio_demo{
        compatible = "firefly,rk3572-gpio";
        status = "okay";
        firefly-gpio = <&gpio1 RK_PC7 GPIO_ACTIVE_HIGH>;    /* GPIO1_C7_d */
    };
};
```

`GPIO_ACTIVE_HIGH` means active high. To make it active low, change it to `GPIO_ACTIVE_LOW`. This property will be read by the driver.

In the kernel driver, request the pin through the GPIO descriptor interface:

```c
#include <linux/gpio/consumer.h>

struct gpio_desc *gpiod;
gpiod = gpiod_get(&pdev->dev, "demo", GPIOD_OUT_LOW);
gpiod_set_value(gpiod, 1);
gpiod_put(gpiod);
```

You can also use the legacy GPIO number interface to read the GPIO number and flags from the device tree:

```c
#include <linux/gpio.h>
#include <linux/of_gpio.h>

enum of_gpio_flags {
	OF_GPIO_ACTIVE_LOW = 0x1,
};
int of_get_named_gpio_flags(struct device_node *np, const char *propname,
int index, enum of_gpio_flags *flags);
int gpio_is_valid(int gpio);
int gpio_request(unsigned gpio, const char *label);
void gpio_free(unsigned gpio);
int gpio_direction_input(int gpio);
int gpio_direction_output(int gpio, int v);
```

In practice, to read a GPIO you need to set it to input mode first and then read the value:

```c
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

## Interrupt [step]

The interrupt usage of a GPIO is similar to its input/output usage. Add the interrupt resource description in the DTS file:

```text
kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-demo.dtsi
gpio {
	compatible = "firefly-gpio";
	firefly-irq-gpio = <&gpio1 RK_PC7 IRQ_TYPE_EDGE_RISING>;  /* GPIO1_C7_d */
};
```

`IRQ_TYPE_EDGE_RISING` means the interrupt is triggered on the rising edge. It can also be configured as:

```text
IRQ_TYPE_NONE         // default value, undefined trigger type
IRQ_TYPE_EDGE_RISING  // rising edge trigger
IRQ_TYPE_EDGE_FALLING // falling edge trigger
IRQ_TYPE_EDGE_BOTH    // both edges trigger
IRQ_TYPE_LEVEL_HIGH   // high level trigger
IRQ_TYPE_LEVEL_LOW    // low level trigger
```

Then parse the resources added in the DTS in the probe function, and register the interrupt:

```c
static int firefly_gpio_probe(struct platform_device *pdev)
{
	int ret;
	int gpio;
	enum of_gpio_flags flag;
	struct firefly_gpio_info *gpio_info;
	struct device_node *firefly_gpio_node = pdev->dev.of_node;
	...

	gpio_info->firefly_irq_gpio = gpio;
	gpio_info->firefly_irq_mode = flag;
	gpio_info->firefly_irq = gpio_to_irq(gpio_info->firefly_irq_gpio);
	if (gpio_info->firefly_irq)
	{
		if (gpio_request(gpio, "firefly-irq-gpio"))
		{
			dev_err(&pdev->dev, "firefly-irq-gpio: %d request failed!\n", gpio);
			gpio_free(gpio);
			return IRQ_NONE;
		}

		ret = request_irq(gpio_info->firefly_irq, firefly_gpio_irq,
							flag, "firefly-gpio", gpio_info);
		if (ret != 0)
		{
			free_irq(gpio_info->firefly_irq, gpio_info);
			dev_err(&pdev->dev, "Failed to request IRQ: %d\n", ret);
		}
	}
	printk("Firefly irq gpio finish \n");
	return 0;
}

static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) // interrupt handler
{
	printk("Enter firefly gpio irq test program!\n");
	return IRQ_HANDLED;
}
```

`gpio_to_irq` converts the GPIO pin value to the corresponding IRQ value, `gpio_request` requests the IO port, and `request_irq` requests the interrupt. If it fails, call `free_irq` to release it. In this function, `firefly_irq` is the hardware interrupt number to request, `firefly_gpio_irq` is the interrupt handler, `firefly_irq_mode` is the interrupt handling attribute, `firefly-gpio` is the device driver name, and `gpio_info` is the `device` structure of the device, used when registering a shared interrupt.

## Multiplexing [step]

`This case is for reference only; the actual hardware interface prevails`

Besides general-purpose input/output and interrupt functions, a GPIO may have other multiplexed functions. Taking `I2C2` on the J1 header as an example, its pins have both GPIO and I2C functions. Check `/d/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins` to confirm the current function of each pin. If the pin has been multiplexed as I2C, disable the corresponding node in the DTS before using it as GPIO:

```text
&i2c2 {
    status = "disabled";
};
```

Switching the pin function at runtime relies on the kernel pinctrl framework: the `pinctrl-names` property of the controller node defines the list of state names (such as the default function state and the gpio state), and `pinctrl-0`, `pinctrl-1` define the pin multiplexing configurations of each state. The driver probe flow selects the pinctrl state set in the DTS through `pinctrl_select_state`. For details, refer to `Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf` under `SDK/docs`.

## Debug Methods [step]

### GPIO Debug Interface

The Debugfs filesystem provides developers with more kernel data for debugging. The GPIO interface in Debugfs is `/sys/kernel/debug/gpio`:

```shell
:/ # cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-31, parent: platform/2002c000.gpio, gpio0:
 ...
gpiochip1: GPIOs 32-63, parent: platform/20030000.gpio, gpio1:
 gpio-55  (                    |firefly-gpio        ) out hi
 ...
```

(The output above is a reference example; the actual content depends on the running system.) From the information you can see the current state of all GPIOs listed by the kernel, for example gpio-55 (GPIO1_C7) outputting high (out hi).

### Check pinmux-pins

```shell
:/ # cat /d/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins
```

The result (reference example):

```text
Pinmux settings per pin
Format: pin (name): mux_owner gpio_owner hog?
pin 0 (gpio0-0): wireless-bluetooth gpio0:0 function wireless-bluetooth group bt-irq-gpio
pin 1 (gpio0-1): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 2 (gpio0-2): (MUX UNCLAIMED) (GPIO UNCLAIMED)
...
```

Explanation: the `pin 0` column is the pin number, the `gpio0-0` column is the gpio group number, the `MUX UNCLAIMED` column is the owner of the mux, and the `GPIO UNCLAIMED` column is the owner of the gpio.

`MUX UNCLAIMED` means the pin has not been controlled by pinctrl of any node. For example, after the i2c2 node is enabled, it owns the pinctrl-0 property, changes the pin function to i2c, and the pin information becomes the state used by the i2c node; after registering the pin with the gpio_demo example, the pin information becomes the pinctrl configuration used by the node named gpio_demo, and the pin is also requested as a gpio.

## FAQs [step]

### Q1: How to switch the MUX value of a PIN back to general GPIO?

A1: When GPIO request is used, the MUX value of the pin will be forcibly switched to GPIO. So make sure the pin is not used by other modules when using it as a GPIO.

### Q2: Why is the value I read with the IO command always 0x00000000?

A2: If the value read from a GPIO register with the IO command is abnormal, such as 0x00000000 or 0xffffffff, please check whether the CLK of the GPIO is turned off. The GPIO CLK is controlled by the CRU. You can check whether the CLK is enabled by reading the CRU_CLKGATE_CON* registers in the datasheet. If it is not enabled, use the io command to set the corresponding register to turn on the CLK. After the CLK is turned on, the correct register value should be readable.

### Q3: What should I check when the measured voltage of a PIN is incorrect?

A3: When the measured voltage of the pin is incorrect, after excluding external factors, confirm whether the IO voltage source of the pin is correct, and whether the IO-Domain configuration is correct.

### Q4: What is the difference between gpio_set_value() and gpio_direction_output()?

A4: If the GPIO will not dynamically switch between input and output, it is recommended to set the GPIO direction at the beginning and use gpio_set_value() to pull it high or low afterwards. It is not recommended to use gpio_direction_output(), because gpio_direction_output contains a mutex lock, which can cause errors in interrupt context, and gpio_direction_output does more work than gpio_set_value, which is wasteful.
