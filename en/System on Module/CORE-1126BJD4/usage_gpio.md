# GPIO

## Introduction

GPIO (General-Purpose Input/Output) is a General pin that can be dynamically configured and controlled during software operation.The initial state of all GPIOs after power-on is input mode, which can be set as pull-up or pull-down or interrupt pin by software. The driving intensity is programmable, and the core of which is to fill the methods and parameters of GPIO bank and register them in the kernel by calling gpiochip_add.


## GPIO Pin to calculate
AIO-1126BJD4 have 7  GPIO bank：GPIO0~GPIO7，Each group was numbered A0~A7, B0~B7, C0~C7, and D0~D7, the following formulas are often used to calculate GPIO Pin:

```
GPIO pin calculation formula：pin = bank * 32 + number 

GPIO group number calculation formula：number = group * 8 + X
```
![](../../../rv1126b_img/CORE-1126BJD4/usage_gpio.png)
The following demonstrates the GPIO1_B7 pin calculation method:

bank = 1;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO<font color=red>1</font>_B7 => 1,  bank ∈ [0,7]

group = B;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO1_<font color=red>B</font>7 => 1,  group ∈ {(A=0), (B=1), (C=2), (D=3)}

X = 7;    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO1_B<font color=red>7</font> => 7, X ∈ [0,7]

number = group * 8 + X = 1 * 8 + 7 = 15

pin = bank * 32 + number= 1 * 32 + 15 = 47;

GPIO1_B7 对应的设备树属性描述为:<&gpio1 9 GPIO_ACTIVE_HIGH>,由`kernel/include/dt-bindings/pinctrl/rockchip.h`的宏定义可知，也可以将 GPIO1_B7 描述为<&gpio1 RK_PB7 GPIO_ACTIVE_HIGH>。
The device tree attribute description corresponding to GPIO1_B7 is: <&gpio1 9 GPIO_ACTIVE_HIGH>. As can be seen from the macro definition in `kernel/include/dt-bindings/pinctrl/rockchip.h`, GPIO1_B7 can also be described as <&gpio1 RK_PB7 GPIO_ACTIVE_HIGH>.

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
When the GPIO1_B7 pin is not multiplexed by other peripherals, we can export the pin for use.

```shell
$ ls /sys/class/gpio/
export  gpiochip0  gpiochip128  gpiochip160  gpiochip192  gpiochip224  gpiochip300  gpiochip32  gpiochip64  gpiochip96  unexport
$ echo 47 > /sys/class/gpio/export
$ ls /sys/class/gpio/
export  gpio47  gpiochip0  gpiochip128  gpiochip160  gpiochip192  gpiochip224  gpiochip300  gpiochip32  gpiochip64  gpiochip96  unexport
$ ls /sys/class/gpio/gpio47
active_low  device  direction  edge  power  subsystem  uevent  value
```
## Input And Output

### Application Layer Settings

After exporting the pins above, you can directly set the gpio to input mode or output mode

* View the current gpio mode

```shell
$ cat /sys/class/gpio/gpio47/direction
in
```

* Set to input mode

```shell
$ echo in > /sys/class/gpio/gpio47/direction
```

* In input mode, you can know the input level by directly viewing the value of the value file

```shell
$ cat /sys/class/gpio/gpio47/value
0
```

* Set to output mode

```shell
$ echo out > /sys/class/gpio/gpio47/direction
```

* * Set output high level

```shell
$ echo 1 > /sys/class/gpio/gpio47/value
```

* Set output low level

```shell
$ echo 0 > /sys/class/gpio/gpio47/value
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

In addition to general input and output and interrupt functions, GPIO ports may also have other multiplexing functions. Taking GPIO7_A4 as an example, there are several functions as follows:

|  func0  |  func1  |  func2  |
| --- | --- | --- |
| PDM_CLK0_M0 | I2C4_SDA_M3 | UART2_CTSN_M1 |

Check `/sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins` to see the function of each pin. If you find that GPIO7_A4 is multiplexed as I2C4_SDA_M3, you need to turn it off in dts.

## Debugging Method

### IO Instruction

A very useful tool for GPIO debugging is the IO command. The Android system of AIO-1126BJD4 has built-in IO commands by default. Using IO commands, you can read or write the status of each IO port in real time. Here is a brief introduction The use of IO instructions. First check the help of IO instruction:

```
$ io --help
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

### GPIO Debug Interface

The purpose of the Debugfs file system is to provide developers with more kernel data to facilitate debugging. Here GPIO debugging can also use the Debugfs file system to get more kernel information. The interface of GPIO in the Debugfs file system is `/sys/kernel/debug/gpio`, the information of this interface can be read like this:

```
root@firefly:/# cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-31, parent: platform/20600000.gpio, gpio0:
 gpio-0   (                    |vcc5v0-sys          ) out hi
 gpio-6   (                    |vccio_sd            ) out hi
 gpio-17  (                    |pwrctrl             ) out lo

gpiochip1: GPIOs 32-63, parent: platform/21300000.gpio, gpio1:

gpiochip2: GPIOs 64-95, parent: platform/21700000.gpio, gpio2:

gpiochip3: GPIOs 96-127, parent: platform/21e00000.gpio, gpio3:
 gpio-102 (                    |bt_default_rts      ) out lo
 gpio-107 (                    |bt_default_wake_host) in  lo IRQ
 gpio-110 (                    |bt_default_wake     ) out hi
 gpio-111 (                    |bt_default_poweron  ) out hi

gpiochip4: GPIOs 128-159, parent: platform/21800000.gpio, gpio4:

gpiochip5: GPIOs 160-191, parent: platform/21900000.gpio, gpio5:
 gpio-166 (                    |vcc3v3-sd           ) out hi ACTIVE LOW

gpiochip6: GPIOs 192-223, parent: platform/21a00000.gpio, gpio6:
 gpio-198 (                    |:user               ) out lo
 gpio-199 (                    |:power              ) out hi
 gpio-207 (                    |reset               ) out hi ACTIVE LOW
 gpio-209 (                    |hp-det              ) in  lo IRQ
...
```

From the information read, we can know that the kernel lists the current status of GPIO. 

### pinmux-pins
Command: 
```
$ cat /sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins
```

Result:
```
Pinmux settings per pin
Format: pin (name): mux_owner gpio_owner hog?
pin 0 (gpio0-0): vcc5v0-sys gpio0:0 function pwr group vcc-sys-en
pin 1 (gpio0-1): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 2 (gpio0-2): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 3 (gpio0-3): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 4 (gpio0-4): 2-0051 (GPIO UNCLAIMED) function rtc group hym8563-int
pin 5 (gpio0-5): 21d60000.mmc (GPIO UNCLAIMED) function sdmmc0 group sdmmc0-detn-pins
pin 6 (gpio0-6): vccio-sd gpio0:6 function sdcard group sdmmc0-vol-ctrl
pin 7 (gpio0-7): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 8 (gpio0-8): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 9 (gpio0-9): 2-004e (GPIO UNCLAIMED) function usb group usbc0-int
pin 10 (gpio0-10): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 11 (gpio0-11): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 12 (gpio0-12): (MUX UNCLAIMED) (GPIO UNCLAIMED)
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
