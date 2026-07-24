# GPIO

## Introduction

GPIO (General-Purpose Input/Output) is a General pin that can be dynamically configured and controlled during software operation.The initial state of all GPIOs after power-on is input mode, which can be set as pull-up or pull-down or interrupt pin by software. The driving intensity is programmable, and the core of which is to fill the methods and parameters of GPIO bank and register them in the kernel by calling gpiochip_add.

This article uses the two general GPIO ports GPIO0_B4 and GPIO4_D5 as examples to write a simple operation GPIO port driver. The path in the SDK is:

```
kernel/drivers/gpio/gpio-firefly.c
```

The following takes this driver as an example to introduce the operation of GPIO.

GPIO0_B4 and GPIO4_D5 are just examples, it does not mean they are available in board.

## Calculate GPIO Pin Number
AIO-3562JQ have 5  GPIO bank：GPIO0~GPIO4，Each group was numbered A0~A7, B0~B7, C0~C7, and D0~D7, the following formulas are often used to calculate GPIO Pin:

GPIO group number calculation formula：number = group * 8 + X

GPIO pin calculation formula：pin = bank * 32 + number 

For example, if we want to calculate GPIO Pin GPIO4_D5, we could refer to the following step: 

bank = 4;   //GPIO<font color=red>4</font>_D5 => 4

group = 3;  //GPIO4_<font color=red>D</font>5 => 3  {(A=0), (B=1), (C=2), (D=3)}

X = 5;      //GPIO4_D<font color=red>5</font> => 5

number = group * 8 + X = 3 * 8 + 5 = 29

pin = bank*32 + number=  4 * 32 + 29 = 157;

GPIO4_D5 property of dts is described as: <&gpio4 29 IRQ_TYPE_EDGE_RISING>, by `kernel/include/dt-bindings/pinctrl/rockchip.h` macro definition, GPIO4_D5 can also be described as :<&gpio4 RK_PD5 IRQ_TYPE_EDGE_RISING>.

## Use GPIO in Userspace

When GPIO is not occupied by other functions, we can export it for using:
```shell
# echo pin number to export
echo 157 > /sys/class/gpio/export

# it will create a folder named "gpio+number"
ls /sys/class/gpio/
export   gpiochip0    gpiochip255  gpiochip500  gpiochip96
gpio157  gpiochip128  gpiochip32   gpiochip64   unexport

# see the properties of gpio157
ls /sys/class/gpio/gpio157
active_low  device  direction  edge  power  subsystem  uevent  value

# this node indicates the gpio direction
cat /sys/class/gpio/gpio157/direction
in
echo out > /sys/class/gpio/gpio157/direction

# this node indicates the gpio value
cat /sys/class/gpio/gpio157/value
0
echo 1 > /sys/class/gpio/gpio157/value
```

## Use GPIO in Kernel Space

First add the resource description of the driver in the DTS file:

```
gpio_demo: gpio_demo {
            status = "okay";
            compatible = "firefly,rk356x-gpio";
            firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
            firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
            };
```

Here defines a pin as a general output and input port:

```
firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;
```

`GPIO_ACTIVE_HIGH` means high level is active, if you want low level to be active, you can change it to: `GPIO_ACTIVE_LOW`, this attribute will be read by the driver.

Then analyze the resources added by DTS in the probe function, the code is as follows:

```c
static int firefly_gpio_probe(struct platform_device *pdev)
{
    int ret;
    int gpio;
    enum of_gpio_flags flag;
    struct firefly_gpio_info *gpio_info;
    struct device_node *firefly_gpio_node = pdev->dev.of_node;

    printk("Firefly GPIO Test Program Probe\n");
    gpio_info = devm_kzalloc(&pdev->dev,sizeof(struct firefly_gpio_info), GFP_KERNEL);
    if (!gpio_info) {
        return -ENOMEM;
    }
    gpio = of_get_named_gpio_flags(firefly_gpio_node, "firefly-gpio", 0, &flag);
    if (!gpio_is_valid(gpio)) {
        printk("firefly-gpio: %d is invalid\n", gpio); return -ENODEV;
    }
    if (gpio_request(gpio, "firefly-gpio")) {
        printk("gpio %d request failed!\n", gpio);
        gpio_free(gpio);
        return -ENODEV;
    }
    gpio_info->firefly_gpio = gpio;
    gpio_info->gpio_enable_value = (flag == OF_GPIO_ACTIVE_LOW) ? 0:1;
    gpio_direction_output(gpio_info->firefly_gpio, gpio_info->gpio_enable_value);
    printk("Firefly gpio putout\n");
    ...
}
```

`of_get_named_gpio_flags` reads the GPIO configuration numbers and flags of `firefly-gpio` and `firefly-irq-gpio` from the device tree, `gpio_is_valid` judges whether the GPIO number is valid, and `gpio_request` applies to occupy the GPIO. If there is an error in the initialization process, you need to call `gpio_free` to release the previously applied and successful GPIO. Call `gpio_direction_output` in the driver to set the output high or low level. Here the default output is the active level `GPIO_ACTIVE_HIGH` obtained from DTS, which is high level. If the drive works normally, you can use a multimeter to measure the corresponding The pin should be high. In practice, if you want to read GPIO, you need to set it to input mode first, and then read the value:

```c
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

The following are commonly used GPIO API definitions:

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

## Interrupt

The Firefly example program also contains an interrupt pin. The interrupt usage of the GPIO port is similar to the input and output of GPIO. First, add the resource description of the driver in the DTS file:

```
kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
gpio {
  compatible = "firefly-gpio";
  firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
};
```

IRQ_TYPE_EDGE_RISING indicates that the interrupt is triggered by the rising edge, and the interrupt function can be triggered when the pin receives the rising edge signal.This can also be configured as follows：

```c
IRQ_TYPE_NONE			//Default value, no defined interrupt trigger type
IRQ_TYPE_EDGE_RISING	//Rising edge trigger
IRQ_TYPE_EDGE_FALLING	//Falling edge trigger
IRQ_TYPE_EDGE_BOTH		//Trigger on both rising and falling edges
IRQ_TYPE_LEVEL_HIGH		//High level trigger
IRQ_TYPE_LEVEL_LOW		//Low level trigger
```

Then analyze the resources added by DTS in the probe function, and then apply for interrupted registration, the code is as follows:

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
   	if (gpio_info->firefly_irq) {
       		if (gpio_request(gpio, "firefly-irq-gpio")) {
          	printk("gpio %d request failed!\n", gpio); gpio_free(gpio); return IRQ_NONE;
        }
        ret = request_irq(gpio_info->firefly_irq, firefly_gpio_irq, flag, "firefly-gpio", gpio_info);
        if (ret != 0) free_irq(gpio_info->firefly_irq, gpio_info);
           dev_err(&pdev->dev, "Failed to request IRQ: %d\n", ret);
    	}
    	return 0;
}
static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) //中断函数
{
   	printk("Enter firefly gpio irq test program!\n");
    	return IRQ_HANDLED;
}
```

Call `gpio_to_irq` to convert the PIN value of the GPIO to the corresponding IRQ value, call `gpio_request` to apply for the IO port, call `request_irq` to apply for an interrupt, if it fails, call `free_irq` to release, in this function `gpio_info-firefly_irq` Is the hardware interrupt number to be applied for, `firefly_gpio_irq` is the interrupt function, `gpio_info->firefly_irq_mode` is the attribute of interrupt processing, `firefly-gpio` is the name of the device driver, and `gpio_info` is the `device` structure of the device. It is used when registering shared interrupts.

## Multiplex

Besides general input/output and interrupt functions, GPIO port may also have other functions, such as GPIO0_B4, it can be used for I2C1_SDA_M0 except GPIO.

When using GPIO port, it is necessary to pay attention to whether it is reused for other functions. 

For example, you need to disable i2c1 when you want to use GPIO0_B4 as GPIO.
```
&i2c1 {
    status = "disabled";
};

gpio_demo: gpio_demo {
    status = "okay";
    compatible = "firefly,rk356x-gpio";
    firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
    firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
};
```

The multiplex of GPIO is controled by "pinctrl", in i2c1 node of DTS:
```
i2c1 {
    /omit-if-no-ref/
    i2c1m0_xfer: i2c1m0-xfer {
        rockchip,pins =
            /* i2c1_scl_m0 */
            <0 RK_PB3 1 &pcfg_pull_none_smt>,
            /* i2c1_sda_m0 */
            <0 RK_PB4 1 &pcfg_pull_none_smt>; // Here configed GPIO0_B4 to be func1, which is i2c1_sda_m0
    };
	...
};
```
So if you didn't disable i2c1, then the pinctrl in i2c1 will take effect, GPIO0_B4 will be used for i2c1_sda_m0.

More details about pinctrl please refer to SDK/docs/en/Common/PINCTRL/Rockchip_Developer_Guide_Linux_Pinctrl_EN.pdf

## Debugging method

### IO instruction

A very useful tool for GPIO debugging is the IO command. The firmware of AIO-3562JQ has built-in IO commands by default. Using IO commands, you can read or write the status of each IO port in real time. Here is a brief introduction The use of IO instructions. First check the help of IO instruction:

```
#io --help
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

Use example:

* View the multiplexing of GPIO1_B3 pins
* From the datasheet of the master control, the base address of the register corresponding to GPIO1 is: 0xff320000
* The offset of GPIO1B_IOMUX found from the datasheet of the master control is: 0x00014
* The address of the iomux register of GPIO1_B3 is: base address (Operational Base) + offset (offset)=0xff320000+0x00014=0xff320014
* Use the following command to check the multiplexing of GPIO1_B3:

```
# io -4 -r 0xff320014
ff320014:  0000816a
```

* Find [7:6] from the datasheet:

```
gpio1b3_sel
GPIO1B[3] iomux select
2'b00: gpio
2'b01: i2c4sensor_sda
2'b10: reserved
2'b11: reserved
```

Therefore, it can be determined that the GPIO is multiplexed as i2c4sensor_sda.

* If you want to reuse as GPIO, you can use the following command settings:

```
# io -4 -w 0xff320014 0x0000812a
```

### Debugfs

The purpose of the Debugfs file system is to provide developers with more kernel data to facilitate debugging. Here GPIO debugging can also use the Debugfs file system to get more kernel information. The interface of GPIO in the Debugfs file system is `/sys/kernel/debug/gpio`.

## FAQs

### Q1: How to switch the MUX value of PIN to normal GPIO?

A1: When using GPIO request, the MUX value of the PIN will be forcibly switched to GPIO, so when using the PIN pin as a GPIO function, make sure that the PIN pin is not used by other modules.

### Q2: Why is the value I read out with the IO instruction is 0x00000000?

A2: If you use the IO command to read the register of a GPIO, the value read is abnormal, such as 0x00000000 or 0xffffffff, etc., please confirm whether the CLK of the GPIO is turned off. The CLK of the GPIO is controlled by the CRU. You can read the datasheet Next, use the CRU_CLKGATE_CON* register to check whether the CLK is turned on. If it is not turned on, you can use the io command to set the corresponding register to turn on the corresponding CLK. After turning on the CLK, you should be able to read the correct register value.

### Q3: How to check if the voltage of the PIN pin is wrong?

A3: When measuring the voltage of the PIN pin is incorrect, if external factors are excluded, you can confirm whether the IO voltage source where the PIN is located is correct and whether the IO-Domain configuration is correct.
