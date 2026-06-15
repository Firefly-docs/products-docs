# GPIO

## Introduction

GPIO (General-Purpose Input/Output) is a General pin that can be dynamically configured and controlled during software operation.The initial state of all GPIOs after power-on is input mode, which can be set as pull-up or pull-down or interrupt pin by software. The driving intensity is programmable, and the core of which is to fill the methods and parameters of GPIO bank and register them in the kernel by calling gpiochip_add.




This article uses the two general GPIO ports GPIO0_B4 and GPIO4_D5 as examples to write a simple operation GPIO port driver. The path in the SDK is:

```
kernel/drivers/gpio/gpio-firefly.c
```

The following takes this driver as an example to introduce the operation of GPIO.



## GPIO Pin to calculate
AIO-3568J have 5  GPIO bank：GPIO0~GPIO4，Each group was numbered A0~A7, B0~B7, C0~C7, and D0~D7, the following formulas are often used to calculate GPIO Pin:

```
GPIO pin calculation formula：pin = bank * 32 + number 

GPIO group number calculation formula：number = group * 8 + X
```

For example, if we want to calculate GPIO Pin GPIO4_D5, we could refer to the following step: 

bank = 4;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO<font color=red>4</font>_D5 => 4,  bank ∈ [0,4]

group = 3;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO4_<font color=red>D</font>5 => 3,  group ∈ {(A=0), (B=1), (C=2), (D=3)}

X = 5;    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO4_D<font color=red>5</font> => 5, X ∈ [0,7]

number = group * 8 + X = 3 * 8 + 5 = 29

pin = bank*32 + number=  4 * 32 + 29 = 157;

GPIO4_D5 property of dts is described as :<&gpio4 29 IRQ_TYPE_EDGE_RISING>, by `kernel/include/dt-bindings/pinctrl/rockchip.h` macro definition ，GPIO4_D5 can also be described as :<&gpio4 RK_PD5 IRQ_TYPE_EDGE_RISING>。

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

GPIO4_D5 may be used by other funtion, here is just an example. When GPIO4_D5 pin is not reused by other peripherals, we can export this pin to use
```shell
:/ # ls /sys/class/gpio/
export     gpiochip128  gpiochip32   gpiochip64  unexport
gpiochip0  gpiochip255  gpiochip500  gpiochip96
:/ # echo 157 > /sys/class/gpio/export
:/ # ls /sys/class/gpio/
export   gpiochip0    gpiochip255  gpiochip500  gpiochip96
gpio157  gpiochip128  gpiochip32   gpiochip64   unexport
:/ # ls /sys/class/gpio/gpio157
active_low  device  direction  edge  power  subsystem  uevent  value
:/ # cat /sys/class/gpio/gpio157/direction
in
:/ # cat /sys/class/gpio/gpio157/value
0
```

## input Output

First add the resource description of the driver in the DTS file:

```
kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
gpio_demo: gpio_demo {
            status = "okay";
            compatible = "firefly,rk356x-gpio";
            firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
            firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
            };

```

Here defines a pin as a general output and input port:

```
firefly-gpio GPIO0_B4
```

`GPIO_ACTIVE_HIGH` means high level is active, if you want low level to be active, you can change it to: `GPIO_ACTIVE_LOW`, this attribute will be read by the driver.

Then analyze the resources added by DTS in the probe function, the code is as follows:

```
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

The Firefly example program also contains an interrupt pin. The interrupt usage of the GPIO port is similar to the input and output of GPIO. First, add the resource description of the driver in the DTS file:

```
kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
gpio {
  compatible = "firefly-gpio";
  firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
};
```

IRQ_TYPE_EDGE_RISING indicates that the interrupt is triggered by the rising edge, and the interrupt function can be triggered when the pin receives the rising edge signal.This can also be configured as follows：

```
IRQ_TYPE_NONE			//Default value, no defined interrupt trigger type
IRQ_TYPE_EDGE_RISING	//Rising edge trigger
IRQ_TYPE_EDGE_FALLING	//Falling edge trigger
IRQ_TYPE_EDGE_BOTH		//Trigger on both rising and falling edges
IRQ_TYPE_LEVEL_HIGH		//High level trigger
IRQ_TYPE_LEVEL_LOW		//Low level trigger
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

## Reuse

In addition to general input/output and interrupt functions, GPIO port may also have other reuse functions, such as GPIO0_B4, which has the following functions:

|  func0  |  func1  |  func2  |  func3  |  func4  |
| --- | --- | --- | --- | --- |
| GPIO0_B4 | I2C1_SDA | CAN0_RX_M0 | PCIE20_BUTTONRSTn | MCU_JTAG_TCK |  

When using GPIO port, it is necessary to pay attention to whether it is reused for other functions. Here, you can use IO command to check the IOMUX to judge whether it is reused or not. It is explained in the section of Debugging Methods, but it will not be explained here.
If GPIO0_B4 is found to be reused as I2C1_SDA through IO command, the I2C1 disabled shall be removed when GPIO0_B4 is used as GPIO or other functions.  

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
**Note:** This GPIO0_B4 is for example only and is not recommended for practical use

The above mentioned changes on DTS, but how do you switch functions at run time?The following take I2C4_M0 as an example for a simple introduction, detailed introduction can be referred to `RKDocs/common/PIN-Ctrl/Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf`.

According to the specification table, the functions of I2C4_SDA_M0 and I2C4_SCL_M0 are defined as follows:

|  Pad#  |  func0  |  func1  |  func2  |  func3  |  func4  | func5 |
| --- | --- | --- | --- | --- | --- | --- |
| I2C4_SDA_M0/GPIO4_B2 | GPIO4_B2 | I2C4_SDA | EBC_VCOM | GMAC1_RXER_M1 | SPI3_MOSI_M0 | I2S2_SDI_M1 |
| I2C4_SCL_M0/GPIO4_B3 | GPIO4_B3 | I2C4_SDA | EBC_GDOE | ETH1_REFCLKO_25MM1 | SPI3_CLK_M0 | I2S2_SDO_M1 |

In `kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi` there are:

```
i2c4: i2c@fe5d0000 {
  compatible = "rockchip,rk3399-i2c";
  reg = <0x0 0xfe5d0000 0x0 0x1000>;
  clocks = <&cru CLK_I2C4>, <&cru PCLK_I2C4>;
  clock-names = "i2c", "pclk";
  interrupts = <GIC_SPI 50 IRQ_TYPE_LEVEL_HIGH>;
  pinctrl-names = "default", "gpio";
  pinctrl-0 = <&i2c4m0_xfer>;
  pinctrl-1 = <&i2c4mo_gpio>;  //此处源码未添加
  #address-cells = <1>;
  #size-cells = <0>;
  status = "disabled";
};
```

Related to multiplexing control is the attribute at the beginning of `pinctrl-`:

* pinctrl-names defines a list of state names: default (i2c function) and gpio two states.
* pinctrl-0 defines the pinctrl that needs to be set in state 0 (ie default): &i2c4m0_xfer
* pinctrl-1 defines the pinctrl that needs to be set in state 1 (i.e. gpio): &i2c4mo_gpio

These pinctrls are defined in `kernel/arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi` as follows:

```
pinctrl: pinctrl {
  compatible = "rockchip,rk3568-pinctrl";
  rockchip,grf = <&grf>;
  rockchip,pmu = <&pmugrf>;
  #address-cells = <2>;
  #size-cells = <2>;
  ranges;

  i2c4 {
    /omit-if-no-ref/
    i2c4m0_xfer: i2c4m0-xfer {
      rockchip,pins =
        /* i2c4_sclm0 */
        <4 RK_PB3 1 &pcfg_pull_none_smt>,
        /* i2c4_sdam0 */
        <4 RK_PB2 1 &pcfg_pull_none_smt>;
    };
  };
  // Source code is not added here, just for example
  gpio {
    /omit-if-no-ref/
    i2c4m0_gpio: i2c4m0-gpio {
      rockchip,pins =
        <4 RK_PB2 0 &pcfg_pull_none>,
        <4 RK_PB3 0 &pcfg_pull_none>;
        };
  };
};
```

RK_FUNC_1,RK_FUNC_GPIO is defined in  `kernel/include/dt-bindings/pinctrl/rockchip.h`,Here they are simply written as 0 and 1：

```
 #define RK_FUNC_GPIO    0
 #define RK_FUNC_1   1
 #define RK_FUNC_2   2
 #define RK_FUNC_3   3
 #define RK_FUNC_4   4
 #define RK_FUNC_5   5
 #define RK_FUNC_6   6
 #define RK_FUNC_7   7
```

During reuse, if 'default' (i.e. I2C function) is selected, the system will apply the pinctrl of i2c4m0_xfer, and finally switch the pins of GPIO4_B2 and GPIO4_B3 to the corresponding I2C function;If 'gpio' is selected, the system will apply the pinctrl of i2c4m0_gpio to restore the pins of GPIO4_B2 and GPIO4_B3 to GPIO function.

Let's take a look at how the i2c driver switches reuse functions:

```
static int rockchip_i2c_probe(struct platform_device *pdev)
{
	struct rockchip_i2c *i2c = NULL; struct resource *res;
    	struct device_node *np = pdev->dev.of_node; int ret;//
    	 ...
    	i2c->sda_gpio = of_get_gpio(np, 0);
   	 if (!gpio_is_valid(i2c->sda_gpio)) {
		dev_err(&pdev->dev, "sda gpio is invalid\n");
		return -EINVAL;
        }
	ret = devm_gpio_request(&pdev->dev, i2c->sda_gpio, dev_name(&i2c->adap.dev));
	if (ret) {
    		dev_err(&pdev->dev, "failed to request sda gpio\n");
		return ret;
	}
	i2c->scl_gpio = of_get_gpio(np, 1);
	if (!gpio_is_valid(i2c->scl_gpio)) {
		dev_err(&pdev->dev, "scl gpio is invalid\n");
		return -EINVAL;
	}
	ret = devm_gpio_request(&pdev->dev, i2c->scl_gpio, dev_name(&i2c->adap.dev));
	if (ret) {
		dev_err(&pdev->dev, "failed to request scl gpio\n");
		return ret;
	}
	i2c->gpio_state = pinctrl_lookup_state(i2c->dev->pins->p, "gpio");
	if (IS_ERR(i2c->gpio_state)) {
		dev_err(&pdev->dev, "no gpio pinctrl state\n");
		return PTR_ERR(i2c->gpio_state);
    	}
	pinctrl_select_state(i2c->dev->pins->p, i2c->gpio_state);
	gpio_direction_input(i2c->sda_gpio);
	gpio_direction_input(i2c->scl_gpio);
	pinctrl_select_state(i2c->dev->pins->p, i2c->dev->pins->default_state);
	...
}
```

The first is to call of_get_gpio to take out the gpios of the i2c4 node in the device tree, which belongs to the two defined gpio:

```
gpios = <&gpio4 GPIO_B2 GPIO_ACTIVE_LOW>, <&gpio4 GPIO_B3 GPIO_ACTIVE_LOW>;
```

Then devm_gpio_request is called to apply for gpio, and then pinctrl_lookup_state is called to find the gpio state, and the default state `default` has been saved by the framework to i2c->dev-pins->default_state.

Finally, call pinctrl_select_state to select the `default` or `gpio` function.

The following are commonly used multiplexing API definitions:

```
#include <linux/pinctrl/consumer.h>
struct device {
	//...
	#ifdef CONFIG_PINCTRL
	struct dev_pin_info	*pins;
	#endif
	//...
};
struct dev_pin_info {
    	struct pinctrl *p;
    	struct pinctrl_state *default_state;
#ifdef CONFIG_PM
   	struct pinctrl_state *sleep_state;
   	struct pinctrl_state *idle_state;
#endif
};
struct pinctrl_state * pinctrl_lookup_state(struct pinctrl *p, const char *name);
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
```
## FAQs

### Q1: How to switch the MUX value of PIN to normal GPIO?

A1: When using GPIO request, the MUX value of the PIN will be forcibly switched to GPIO, so when using the PIN pin as a GPIO function, make sure that the PIN pin is not used by other modules.

### Q2: Why is the value I read out with the IO instruction is 0x00000000?

A2: If you use the IO command to read the register of a GPIO, the value read is abnormal, such as 0x00000000 or 0xffffffff, etc., please confirm whether the CLK of the GPIO is turned off. The CLK of the GPIO is controlled by the CRU. You can read the datasheet Next, use the CRU_CLKGATE_CON* register to check whether the CLK is turned on. If it is not turned on, you can use the io command to set the corresponding register to turn on the corresponding CLK. After turning on the CLK, you should be able to read the correct register value.

### Q3: How to check if the voltage of the PIN pin is wrong?

A3: When measuring the voltage of the PIN pin is incorrect, if external factors are excluded, you can confirm whether the IO voltage source where the PIN is located is correct and whether the IO-Domain configuration is correct.

### Q4: What is the difference between gpio_set_value() and gpio_direction_output()?

A4: If you do not dynamically switch input and output when using this GPIO, it is recommended to set the GPIO output direction at the beginning, and use the gpio_set_value() interface when pulling it up and pulling it down later. It is not recommended to use gpio_direction_output() because of the gpio_direction_output interface There is a mutex lock inside, there will be an error exception when calling the interrupt context, and compared to gpio_set_value, gpio_direction_output does more and is wasteful.
