# GPIO 使用

## 简介

GPIO, 全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。 所有的 GPIO 在上电后的初始状态都是输入模式，可以通过软件设为上拉或下拉，也可以设置为中断脚，驱动强度都是可编程的,其核心是填充 GPIO bank 的方法和参数，并调用 gpiochip_add 注册到内核中。

本文以 GPIO0_B4 和 GPIO4_D5 这两个 GPIO 口为例写了一份简单操作 GPIO 口的驱动，在 SDK 的路径为：`kernel/drivers/gpio/gpio-firefly.c`,以下就以该驱动为例介绍 GPIO 的操作。

## GPIO引脚计算
IPC-M10R800-A3568J 有 5 组 GPIO bank：GPIO0~GPIO4，每组又以 A0~A7, B0~B7, C0~C7, D0~D7 作为编号区分,常用以下公式计算引脚：

```shell
GPIO pin脚计算公式：pin = bank * 32 + number

GPIO 小组编号计算公式：number = group * 8 + X
```

下面演示GPIO4_D5 pin脚计算方法：

bank = 4;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO<font color=red>4</font>_D5 => 4,  bank ∈ [0,4]

group = 3;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO4_<font color=red>D</font>5 => 3,  group ∈ {(A=0), (B=1), (C=2), (D=3)}

X = 5;    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO4_D<font color=red>5</font> => 5, X ∈ [0,7]

number = group * 8 + X = 3 * 8 + 5 = 29

pin = bank*32 + number=  4 * 32 + 29 = 157;

GPIO4_D5 对应的设备树属性描述为:<&gpio4 29 IRQ_TYPE_EDGE_RISING>,由`kernel/include/dt-bindings/pinctrl/rockchip.h`的宏定义可知，也可以将GPIO4_D5描述为<&gpio4 RK_PD5 IRQ_TYPE_EDGE_RISING>。

```
#define RK_PA0		0
#define RK_PA1		1
#define RK_PA2		2
#define RK_PA3		3
#define RK_PA4		4
#define RK_PA5		5
#define RK_PA6		6
#define RK_PA7		7
#define RK_PB0		8
#define RK_PB1		9
#define RK_PB2		10
#define RK_PB3		11
......
```

GPIO4_D5 可能被其他功能占用，以下仅是举例说明。当引脚没有被其它外设复用时, 我们可以通过export导出该引脚去使用
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

## 输入输出

首先在 DTS 文件中增加驱动的资源描述：

```
kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
gpio_demo: gpio_demo {
            status = "okay";
            compatible = "firefly,rk356x-gpio";
            firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
            firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
            };
```

这里定义了一个脚作为一般的输出输入口：

```
firefly-gpio GPIO0_B4
```

`GPIO_ACTIVE_HIGH` 表示高电平有效，如果想要低电平有效，可以改为：`GPIO_ACTIVE_LOW`，这个属性将被驱动所读取。

然后在 probe 函数中对 DTS 所添加的资源进行解析，代码如下：

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

`of_get_named_gpio_flags` 从设备树中读取 `firefly-gpio` 和 `firefly-irq-gpio` 的 GPIO 配置编号和标志，`gpio_is_valid` 判断该 GPIO 编号是否有效，`gpio_request` 则申请占用该 GPIO。如果初始化过程出错，需要调用 `gpio_free` 来释放之前申请过且成功的 GPIO 。在驱动中调用 `gpio_direction_output` 就可以设置输出高还是低电平，这里默认输出从 DTS 获取得到的有效电平 `GPIO_ACTIVE_HIGH`，即为高电平，如果驱动正常工作，可以用万用表测得对应的引脚应该为高电平。实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

```
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

下面是常用的 GPIO API 定义：

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

## 中断

在 Firefly 的例子程序中还包含了一个中断引脚，GPIO 口的中断使用与 GPIO 的输入输出类似，首先在 DTS 文件中增加驱动的资源描述：

```
kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi
gpio {
	compatible = "firefly-gpio";
	firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
};
```

IRQ_TYPE_EDGE_RISING 表示中断由上升沿触发，当该引脚接收到上升沿信号时可以触发中断函数。 这里还可以配置成如下：

```
IRQ_TYPE_NONE         //默认值，无定义中断触发类型
IRQ_TYPE_EDGE_RISING  //上升沿触发
IRQ_TYPE_EDGE_FALLING //下降沿触发
IRQ_TYPE_EDGE_BOTH    //上升沿和下降沿都触发
IRQ_TYPE_LEVEL_HIGH   //高电平触发
IRQ_TYPE_LEVEL_LOW    //低电平触发
```

然后在 probe 函数中对 DTS 所添加的资源进行解析，再做中断的注册申请，代码如下：

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

调用 `gpio_to_irq` 把 GPIO 的 PIN 值转换为相应的 IRQ 值，调用 `gpio_request` 申请占用该 IO 口，调用 `request_irq` 申请中断，如果失败要调用 `free_irq` 释放，该函数中 `gpio_info-firefly_irq` 是要申请的硬件中断号，`firefly_gpio_irq` 是中断函数，`gpio_info->firefly_irq_mode` 是中断处理的属性，`firefly-gpio` 是设备驱动程序名称，`gpio_info` 是该设备的 `device` 结构，在注册共享中断时会用到。

## 复用

GPIO 口除了通用输入输出、中断功能外，还可能有其它复用功能，如GPIO0_B4, 就有如下几个功能：

|  func0  |  func1  |  func2  |  func3  |  func4  |
| --- | --- | --- | --- | --- |
| GPIO0_B4 | I2C1_SDA | CAN0_RX_M0 | PCIE20_BUTTONRSTn | MCU_JTAG_TCK |

那么在使用作GPIO口时，就需要注意是否被复用为其他功能了，这里可以用io命令查看iomux来判断是否复用，在调试方法的章节中有相关说明，这里不多作说明。
假如通过io命令发现GPIO0_B4有复用作I2C1_SDA，使用GPIO0_B4作gpio或者其他功能时就需要将I2C1 disabled掉。

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
**Note:** 此处GPIO0_B4仅作示例，实际使用中不推荐如此修改

上面介绍了在DTS上修改，那在运行时又如何切换功能呢？下面以 I2C4_m0 为例作简单的介绍,详细介绍可以参考`RKDocs/common/PIN-Ctrl/Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf`。

查规格表可知，I2C4_SDA_M0 与 I2C4_SCL_M0 的功能定义如下：

|  Pad#  |  func0  |  func1  |  func2  |  func3  |  func4  | func5 |
| --- | --- | --- | --- | --- | --- | --- |
| I2C4_SDA_M0/GPIO4_B2 | GPIO4_B2 | I2C4_SDA | EBC_VCOM | GMAC1_RXER_M1 | SPI3_MOSI_M0 | I2S2_SDI_M1 |
| I2C4_SCL_M0/GPIO4_B3 | GPIO4_B3 | I2C4_SDA | EBC_GDOE | ETH1_REFCLKO_25MM1 | SPI3_CLK_M0 | I2S2_SDO_M1 |

在 `kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi` 里有：

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

跟复用控制相关的是 `pinctrl-` 开头的属性：

* pinctrl-names 定义了状态名称列表： default (i2c 功能) 和 gpio 两种状态。
* pinctrl-0 定义了状态 0 (即 default）时需要设置的 pinctrl: &i2c4m0_xfer
* pinctrl-1 定义了状态 1 (即 gpio)时需要设置的 pinctrl: &i2c4mo_gpio

这些 pinctrl 在 `kernel/arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi` 中这样定义：

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
	// 此处源码未添加，仅作示例
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

RK_FUNC_1,RK_FUNC_GPIO 的定义在 `kernel/include/dt-bindings/pinctrl/rockchip.h` ，此处简写作0和1：

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

在复用时，如果选择了 `default` （即 i2c 功能),系统会应用 i2c4m0_xfer 这个 pinctrl，最终将 GPIO4_B2 和 GPIO4_B3 两个针脚切换成对应的 i2c 功能；而如果选择了 `gpio` ，系统会应用 i2c4m0_gpio 这个 pinctrl，将 GPIO4_B2 和 GPIO4_B3 两个针脚还原为 GPIO 功能。

我们看看如下 i2c 的驱动程序是如何切换复用功能的：

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

首先是调用 of_get_gpio 取出设备树中 i2c4 结点的 gpios 属于所定义的两个 gpio:

```
gpios = <&gpio1 GPIO_B3 GPIO_ACTIVE_LOW>, <&gpio1 GPIO_B4 GPIO_ACTIVE_LOW>;
```

然后是调用 devm_gpio_request 来申请 gpio，接着是调用 pinctrl_lookup_state 来查找 gpio 状态，而默认状态 `default` 已经由框架保存到 i2c->dev-pins->default_state 中了。

最后调用 pinctrl_select_state 来选择是 `default` 还是 `gpio` 功能。

下面是常用的复用 API 定义：

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

### Q1: 如何将 PIN 的 MUX 值切换为一般的 GPIO？

A1: 当使用 GPIO request 时候，会将该 PIN 的 MUX 值强制切换为 GPIO，所以使用该 PIN 脚为 GPIO 功能的时候确保该 PIN 脚没有被其他模块所使用。

### Q2: 为什么我用 IO 指令读出来的值都是 0x00000000？

A2: 如果用 IO 命令读某个 GPIO 的寄存器，读出来的值异常,如 0x00000000 或 0xffffffff 等，请确认该 GPIO 的 CLK 是不是被关了，GPIO 的 CLK 是由 CRU 控制，可以通过读取 datasheet 下面 CRU_CLKGATE_CON* 寄存器来查到 CLK 是否开启，如果没有开启可以用 io 命令设置对应的寄存器，从而打开对应的 CLK，打开 CLK 之后应该就可以读到正确的寄存器值了。

### Q3: 测量到 PIN 脚的电压不对应该怎么查？

A3: 测量该 PIN 脚的电压不对时，如果排除了外部因素，可以确认下该 PIN 所在的 IO 电压源是否正确，以及 IO-Domain 配置是否正确。

### Q4: gpio_set_value() 与 gpio_direction_output() 有什么区别？

A4: 如果使用该 GPIO 时，不会动态的切换输入输出，建议在开始时就设置好 GPIO 输出方向，后面拉高拉低时使用 gpio_set_value() 接口，而不建议使用 gpio_direction_output(), 因为 gpio_direction_output 接口里面有 mutex 锁，对中断上下文调用会有错误异常，且相比 gpio_set_value，gpio_direction_output 所做事情更多，浪费。
