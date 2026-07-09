# GPIO

## Introduction

GPIO (General-Purpose Input/Output) is a General pin that can be dynamically configured and controlled during software operation.The initial state of all GPIOs after power-on is input mode, which can be set as pull-up or pull-down or interrupt pin by software. The driving intensity is programmable, and the core of which is to fill the methods and parameters of GPIO bank and register them in the kernel by calling gpiochip_add.



## GPIO Pin to calculate
C40PL have 5  GPIO bank：GPIO0~GPIO4，Each group was numbered A0~A7, B0~B7, C0~C7, and D0~D7, the following formulas are often used to calculate GPIO Pin:

```
GPIO pin calculation formula：pin = bank * 32 + number 

GPIO group number calculation formula：number = group * 8 + X
```

```
#define RK_PA0    0
#define RK_PA1    1
#define RK_PA2    2
#define RK_PA3    3
#define RK_PA4    4
#define RK_PA5    5
#define RK_PA6    6
#define RK_PA7    7
#define RK_PB0    8
#define RK_PB1    9
#define RK_PB2    10
#define RK_PB3    11
......
```
When the GPIO2_C0 pin is not reused by other peripherals, we can export the pin to use it.

```shell
[root@RV1126_RV1109:/]# ls /sys/class/gpio/
export  gpiochip0  gpiochip128  gpiochip32  gpiochip503  gpiochip511  gpiochip64  gpiochip96  unexport
[root@RV1126_RV1109:/]# echo 80 > /sys/class/gpio/export
[root@RV1126_RV1109:/]# ls /sys/class/gpio/
export  gpio80  gpiochip0  gpiochip128  gpiochip32  gpiochip503  gpiochip511  gpiochip64  gpiochip96  unexport
[root@RV1126_RV1109:/]# ls /sys/class/gpio/gpio80
active_low  device  direction  edge  power  subsystem  uevent  value
```

## Input And Output

### Application Layer Settings

After exporting the pins above, you can directly set the gpio to input mode or output mode

* View the current gpio mode
```shell
[root@RV1126_RV1109:/]# cat /sys/class/gpio/gpio80/direction
in
```

* Set to input mode

```shell
[root@RV1126_RV1109:/]# echo in > /sys/class/gpio/gpio80/direction
```

* In input mode, directly view the value of the value file to know the input level

```shell
[root@RV1126_RV1109:/]# cat /sys/class/gpio/gpio80/value
0
```

* Set to output mode

```shell
[root@RV1126_RV1109:/]# echo out > /sys/class/gpio/gpio80/direction
```

* 设置输出高电平

```shell
[root@RV1126_RV1109:/]# echo 1 > /sys/class/gpio/gpio80/value
```

* Set output high level

```shell
[root@RV1126_RV1109:/]# echo 0 > /sys/class/gpio/gpio80/value
```




### Driver Layer Settings

`of_get_named_gpio_flags` reads the GPIO configuration numbers and flags of `firefly-gpio` and `firefly-irq-gpio` from the device tree, `gpio_is_valid` judges whether the GPIO number is valid, and `gpio_request` applies to occupy the GPIO. If there is an error in the initialization process, you need to call `gpio_free` to release the previously applied and successful GPIO. Call `gpio_direction_output` in the driver to set the output high or low level. Here the default output is the active level `GPIO_ACTIVE_HIGH` obtained from DTS, which is high level. If the drive works normally, you can use a multimeter to measure the corresponding The pin should be high. In practice, if you want to read GPIO, you need to set it to input mode first, and then read the value:

```
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

The following are commonly used GPIO API definitions:

```
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

## Interrupt

IRQ_TYPE_EDGE_RISING indicates that the interrupt is triggered by the rising edge, and the interrupt function can be triggered when the pin receives the rising edge signal.This can also be configured as follows：

```
IRQ_TYPE_NONE           //Default value, no defined interrupt trigger type
IRQ_TYPE_EDGE_RISING    //Rising edge trigger
IRQ_TYPE_EDGE_FALLING   //Falling edge trigger
IRQ_TYPE_EDGE_BOTH      //Trigger on both rising and falling edges
IRQ_TYPE_LEVEL_HIGH     //High level trigger
IRQ_TYPE_LEVEL_LOW      //Low level trigger
```

Then analyze the resources added by DTS in the probe function, and then apply for interrupted registration, the code is as follows:

```
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

static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) //interrupt function
{
	printk("Enter firefly gpio irq test program!\n");
	return IRQ_HANDLED;
}
```

Call `gpio_to_irq` to convert the PIN value of the GPIO to the corresponding IRQ value, call `gpio_request` to apply for the IO port, call `request_irq` to apply for an interrupt, if it fails, call `free_irq` to release, in this function `gpio_info-firefly_irq` Is the hardware interrupt number to be applied for, `firefly_gpio_irq` is the interrupt function, `gpio_info->firefly_irq_mode` is the attribute of interrupt processing, `firefly-gpio` is the name of the device driver, and `gpio_info` is the `device` structure of the device. It is used when registering shared interrupts.

## Reuse

`For reference only, the actual hardware interfaces prevail.`

In addition to general input and output and interrupt functions, GPIO ports may also have other multiplexing functions. Taking GPIO0_C3 as an example, there are several functions as follows:

|  func0  |  func1  |  func2  |
| --- | --- | --- |
| GPIO0_C3 | I2C2_SDA | PWM5_M0 |

Check `/sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins` to check the functions of each pin. If you find that GPIO0_C2 and GPIO0_C3 are multiplexed as I2C, you need to turn it off in dts

```
&i2c2 {
    status = "disabled";
};
```
**Note:** Here **GPIO0_C2** is only used as an example. It is not recommended to modify it in actual use

The above introduces the modification on DTS, so how to switch functions at runtime? The following is a brief introduction using I2C2 as an example. For detailed introduction, please refer to `docs/Common/Pin-Ctrl/Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf`.

According to the specification table, the function definitions of I2C2_SDA and I2C2_SCL are as follows:

|  Pad#  |  func0  |  func1  |  func2  |
| --- | --- | --- | --- |
| PWM4_M0 | GPIO0_C2 | I2C2_SCL | PWM4_M0 |
| PWM5_M0 | GPIO0_C3 | I2C2_SDA | PWM4_M0 |

In `kernel/arch/arm/boot/dts/rv1126.dtsi`:

```
i2c2: i2c@ff400000 {
	compatible = "rockchip,rv1126-i2c", "rockchip,rk3399-i2c";
	reg = <0xff400000 0x1000>;
	interrupts = <GIC_SPI 6 IRQ_TYPE_LEVEL_HIGH>;
	#address-cells = <1>;
	#size-cells = <0>;
	rockchip,grf = <&pmugrf>;
	clocks = <&pmucru CLK_I2C2>, <&pmucru PCLK_I2C2>;
	clock-names = "i2c", "pclk";
	pinctrl-names = "default";
	pinctrl-0 = <&i2c2_xfer>;
	status = "disabled";
};
```

The attributes starting with `pinctrl-` are related to multiplexing control:

* pinctrl-names defines a list of state names: default (i2c function) and gpio.
* pinctrl-0 defines the pinctrl that needs to be set when state 0 (i.e. default): &i2c2_xfer
* pinctrl-1 defines the pinctrl that needs to be set when state 1 (i.e. gpio): &i2c2_gpio

These pinctrls are defined in `kernel/arch/arm/boot/dts/rv1126.dtsi` as follows:

```
pinctrl: pinctrl {
	compatible = "rockchip,rv1126-pinctrl";
	rockchip,grf = <&grf>;
	rockchip,pmu = <&pmugrf>;
	#address-cells = <1>;
	#size-cells = <1>;
	ranges;
	...
};
```

In `kernel/arch/arm/boot/dts/rv1126.dtsi`, there is the definition of i2c2

```
i2c2 {
	/omit-if-no-ref/
	i2c2_xfer: i2c2-xfer {
		rockchip,pins =
		/* i2c2_scl */
		<0 RK_PC2 1 &pcfg_pull_none_drv_level_0_smt>,
		/* i2c2_sda */
		<0 RK_PC3 1 &pcfg_pull_none_drv_level_0_smt>;
	};
};
```

RK_FUNC_GPIO is defined in `kernel/include/dt-bindings/pinctrl/rockchip.h`, here it is abbreviated as 0:

```
#define RK_FUNC_GPIO	0
```

After knowing the above definition of i2c2, in the kernel's dts Add gpio resources for i2c2 nodes in the file

```
&i2c2 {
	status = "okay";
	pinctrl-names = "default","i2c2_gpio";
	pinctrl-1 = <&i2c2_gpio>;
	gpios = <&gpio0 RK_PC2 GPIO_ACTIVE_HIGH>,<&gpio0 RK_PC3 GPIO_ACTIVE_HIGH>;
};

&pinctrl {
	i2c2{
		/omit-if-no-ref/
		i2c2_gpio: i2c2-gpio{
			rockchip,pins =
			/* i2c2_gpio0_c2 */
			<0 RK_PC2 0 &pcfg_pull_none>,
			/* i2c2_gpio0_c3 */
			<0 RK_PC3 0 &pcfg_pull_none>; 
		};
	}; 
};
```

The I2C driver registration process is as follows:
```
rk3x_i2c_driver_init
    platform_driver_register
        driver_register
            bus_add_driver
                driver_attach
                    bus_for_each_dev
                        __driver_attach
                            device_driver_attach
                                driver_probe_device
                                    really_probe
                                        pinctrl_bind_pins
                                            pinctrl_select_state
```
`pinctrl_select_state` is a function to select pinctrl in dts. 

## Debugging Method

### IO Instruction

A very useful tool for GPIO debugging is the IO command. The Android system of C40PL has built-in IO commands by default. Using IO commands, you can read or write the status of each IO port in real time. Here is a brief introduction The use of IO instructions. First check the help of IO instruction:

```
[root@RV1126_RV1109:/]# io --help
Unknown option: ?
Raw memory i/o utility - $Revision: 1.5 $

io -v -1|2|4 -r|w [-l <len>] [-f <file>] <addr> [<value>]

    -v         Verbose, asks for confirmation
    -1|2|4     Sets memory access size in bytes (default byte)
    -l <len>   Length in bytes of area to access (defaults to
               one access, or whole file length)
    -r|w       Read from or Write to memory (default read)
    -f <file>  File to write on memory read, or
               to read on memory write
    <addr>     The memory address to access
    <val>      The value to write (implies -w)

Examples:
    io 0x1000                  Reads one byte from 0x1000
    io 0x1000 0x12             Writes 0x12 to location 0x1000
    io -2 -l 8 0x1000          Reads 8 words from 0x1000
    io -r -f dmp -l 100 200    Reads 100 bytes from addr 200 to file
    io -w -f img 0x10000       Writes the whole of file to memory

Note access size (-1|2|4) does not apply to file based accesses.
```

As you can see from the help, if you want to read or write a register, you can use:

```
io -4 -r 0x1000 //Read the value of 4-bit register starting from 0x1000
io -4 -w 0x1000 //Write the value of the 4-bit register from 0x1000
```

Usage example:

* Check the multiplexing of GPIO0_C2 pin
* From the datasheet of the master controller, the base address of the register corresponding to GPIO0 is: 0xFE020000
* From the datasheet of the master controller, the offset of GPIO0D_IOMUX is: 0x0010
* The iomux register address of GPIO0_C2 is: base address (Operational Base) + offset (offset) = 0xFE020000 + 0x0010 = 0xFE020010
* Use the following command to check the multiplexing of GPIO1_D0:

```
[root@RV1126_RV1109:/]# io -4 -r 0xFE020010
fe020010:  00000020
```

* From the trm datasheet [10:8]:

```
gpio0c2_sel
3'h0: GPIO0_C2_d
3'h1: I2C2_SCL
3'h3: PWM4_PIN_M0
Others: Reserved
```

Therefore, it can be determined that the GPIO is multiplexed as GPIO0_C2_d.

* If you want to multiplex as I2C2_SCL, you can use the following command to set:

```
[root@RV1126_RV1109:/]# io -4 -w 0xFE020010 0x1000120
```

### GPIO Debug Interface

The purpose of the Debugfs file system is to provide developers with more kernel data to facilitate debugging. Here GPIO debugging can also use the Debugfs file system to get more kernel information. The interface of GPIO in the Debugfs file system is `/sys/kernel/debug/gpio`, the information of this interface can be read like this:

```
[root@RV1126_RV1109:/]# cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-31, parent: platform/pinctrl, gpio0:
 gpio-2   (                    |gpio_fan            ) out hi
 gpio-4   (                    |firefly:blue:power  ) out lo
 gpio-5   (                    |bt_default_wake_host) in  hi
 gpio-6   (                    |reset               ) out hi
 gpio-7   (                    |bt_default_poweron  ) out hi
 gpio-16  (                    |firefly:yellow:user ) out lo
 gpio-28  (                    |GTP rst gpio        ) in  hi
 gpio-30  (                    |enable              ) out hi

gpiochip1: GPIOs 32-63, parent: platform/pinctrl, gpio1:
 gpio-32  (                    |GTP int gpio        ) in  lo
 gpio-34  (                    |reset               ) out hi
 gpio-35  (                    |enable              ) out hi
 gpio-48  (                    |bt_default_rts      ) out lo
 gpio-52  (                    |vcc5v0_usb2_host    ) out hi
 gpio-53  (                    |hp-det-gpio         ) in  lo
 gpio-54  (                    |vcc_otg_vbus        ) out hi
 gpio-55  (                    |hp-ctl              ) out lo
 gpio-60  (                    |pwdn                ) out lo

gpiochip2: GPIOs 64-95, parent: platform/pinctrl, gpio2:
 gpio-69  (                    |vcc_4g              ) out hi
 gpio-74  (                    |mdio-reset          ) out hi
 ...
```

From the information read, we can know that the kernel lists the current status of GPIO. Taking the GPIO1 group as an example, gpio-55 (GPIO1_C7) outputs a low level (out lo).

### pinmux-pins
Command: 
```
[root@RV1126_RV1109:/]# cat /sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins
```

Result:
```
Pinmux settings per pin
Format: pin (name): mux_owner gpio_owner hog?
pin 0 (gpio0-0): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 1 (gpio0-1): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 2 (gpio0-2): gpio-fan gpio0:2 function gpio_fan group fan-pins
pin 3 (gpio0-3): ffc60000.dwmmc (GPIO UNCLAIMED) function sdmmc0 group sdmmc0-det
pin 4 (gpio0-4): (MUX UNCLAIMED) gpio0:4
pin 5 (gpio0-5): (MUX UNCLAIMED) gpio0:5
pin 6 (gpio0-6): sdio-pwrseq gpio0:6 function sdio-pwrseq group wifi-enable-h
...
```
Analysis:
The column `pin 0` indicates the pin number, the column `gpio0-0` indicates the gpio group number, the column `MUX UNCLAIMED` indicates the owner of the data selector, and the column `GPIO UNCLAIMED` indicates the owner of the gpio.

Among them, `MUX UNCLAIMED` indicates that the pin has not been controlled by the node using pinctrl. `GPIO UNCLAIMED` indicates that no registered gpio uses the pin.

## FAQs

### Q1: How to switch the MUX value of PIN to normal GPIO?

A1: When using GPIO request, the MUX value of the PIN will be forcibly switched to GPIO, so when using the PIN pin as a GPIO function, make sure that the PIN pin is not used by other modules.

### Q2: Why is the value I read out with the IO instruction is 0x00000000?

A2: If you use the IO command to read the register of a GPIO, the value read is abnormal, such as 0x00000000 or 0xffffffff, etc., please confirm whether the CLK of the GPIO is turned off. The CLK of the GPIO is controlled by the CRU. You can read the datasheet Next, use the CRU_CLKGATE_CON* register to check whether the CLK is turned on. If it is not turned on, you can use the io command to set the corresponding register to turn on the corresponding CLK. After turning on the CLK, you should be able to read the correct register value.

### Q3: How to check if the voltage of the PIN pin is wrong?

A3: When measuring the voltage of the PIN pin is incorrect, if external factors are excluded, you can confirm whether the IO voltage source where the PIN is located is correct and whether the IO-Domain configuration is correct.

### Q4: What is the difference between gpio_set_value() and gpio_direction_output()?

A4: If you do not dynamically switch input and output when using this GPIO, it is recommended to set the GPIO output direction at the beginning, and use the gpio_set_value() interface when pulling it up and pulling it down later. It is not recommended to use gpio_direction_output() because of the gpio_direction_output interface There is a mutex lock inside, there will be an error exception when calling the interrupt context, and compared to gpio_set_value, gpio_direction_output does more and is wasteful.
