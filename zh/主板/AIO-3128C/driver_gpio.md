# GPIO 使用

## 简介

GPIO, 全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。

AIO-3128C 有 4 组 GPIO bank： GPIO0，GPIO1, GPIO2, GPIO3。每组又以 A0~A7, B0~B7, C0~C7, D0~D7 作为编号区分。

每个 GPIO 口除了通用输入输出功能外，还可能有其它复用功能，例如 GPIO1_C2，可以复用成以下功能之一：
	
* GPIO1_C2
* SDMMC0_D0
* UART2_TX


每个 GPIO 口的驱动电流、上下拉和重置后的初始状态都不尽相同，详细情况请参考[《RK3128 规格书》](http://www.t-firefly.com/download/fireprime/hardware/RK3128%20Datasheet%20V0.6_20140815.pdf)中的 "RK3128 function IO description" 一章。

AIO-3128C 的 GPIO 驱动是在以下 pinctrl 文件中实现的：

```
kernel/drivers/pinctrl/pinctrl-rockchip.c
```

其核心是填充 GPIO bank 的方法和参数，并调用 gpiochip_add 注册到内核中。

## 使用

开发板有两个电源 LED 灯是 GPIO 口控制的，分别是：

![](../../../rk3128_img/AIO-3128C/Firerprime_leds_schematic.png)

从电路图上看，GPIO 口输出低电平时灯亮，高电平时灯灭。

## 输入输出

下面以电源 LED 灯的驱动为例，讲述如何在内核编写代码控制 GPIO 口的输出。

首先需要在 aio-3128c.dts 中增加驱动的资源描述：
```
firefly-led{
    compatible = "firefly,led";
    led-work = <&gpio1 GPIO_C6 GPIO_ACTIVE_LOW>;
    led-power = <&gpio1 GPIO_C7 GPIO_ACTIVE_LOW>;
    status = "okay";
};
```

这里定义了两颗 LED 灯的 GPIO 设置：

```
led-work  GPIO1_C6  GPIO_ACTIVE_LOW
led-power GPIO1_C7  GPIO_ACTIVE_LOW
```

GPIO_ACTIVE_LOW 表示低电平有效（灯亮），如果是高电平有效，需要替换为 GPIO_ACTIVE_HIGH 。
之后在驱动程序中加入对 GPIO 口的申请和控制则可：

```
#ifdef CONFIG_OF
#include <linux/of.h>
#include <linux/of_gpio.h>
#endif 
static int firefly_led_probe(struct platform_device *pdev) {
	int ret = -1;
    int gpio, flag;
    struct device_node *led_node = pdev->dev.of_node; 
	gpio = of_get_named_gpio_flags(led_node, "led-power", 0, &flag);
    if (!gpio_is_valid(gpio)){
		printk("invalid led-power: %d\n",gpio);
        return -1;
    } 
    if (gpio_request(gpio, "led_power")) {
		printk("gpio %d request failed!\n",gpio);
        return ret;
    }
    led_info.power_gpio = gpio;
    led_info.power_enable_value = (flag == OF_GPIO_ACTIVE_LOW) ? 0 : 1;
    gpio_direction_output(led_info.power_gpio, !(led_info.power_enable_value));
    ...
    on_error:gpio_free(gpio);
}
```

of_get_named_gpio_flags 从设备树中读取 led-power 的 GPIO 配置编号和标志，gpio_is_valid 判断该 GPIO 编号是否有效，gpio_request 则申请占用该 GPIO。如果初始化过程出错，需要调用 gpio_free 来释放之前申请过且成功的 GPIO 。

调用 gpio_direction_output 就可以设置输出高还是低电平，因为是 GPIO_ACTIVE_LOW ，如果要灯亮，需要写入 0 。

实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

```
int val;gpio_direction_input(your_gpio);val = gpio_get_value(your_gpio);
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

## 复用

如何定义 GPIO 有哪些功能可以复用，在运行时又如何切换功能呢？以 I2C1 为例作简单的介绍。  
查规格表可知，I2C1_SDA 与 I2C1_SCL 的功能定义如下：

![](../../../rk3128_img/AIO-3128C/driver_table1.png)

在 /kernel/arch/arm/boot/dts/rk312x.dtsi 里有：

```
i2c1: i2c@20056000 {
    compatible = "rockchip,rk30-i2c";
    reg = <0x20056000 0x1000>;
    interrupts = <GIC_SPI 25 IRQ_TYPE_LEVEL_HIGH>;#address-cells = <1>;#size-cells = <0>;
    pinctrl-names = "default", "gpio";
    pinctrl-0 = <&i2c1_sda &i2c1_scl>;
    pinctrl-1 = <&i2c1_gpio>;
    gpios = <&gpio0 GPIO_A3 GPIO_ACTIVE_LOW>, <&gpio0 GPIO_A2 GPIO_ACTIVE_LOW>;
    clocks = <&clk_gates8 5>;
    rockchip,check-idle = <1>;
    status = "disabled";
};
```

此处，跟复用控制相关的是 pinctrl- 开头的属性：

* pinctrl-names 定义了状态名称列表： default (i2c 功能) 和 gpio 两种状态。
* pinctrl-0 定义了状态 0 (即 default）时需要设置的 pinctrl: i2c1_sda 和 i2c1_scl
* pinctrl-1 定义了状态 1 (即 gpio)时需要设置的 pinctrl: i2c1_gpio


这些 pinctrl 在 /kernel/arch/arm/boot/dts/rk312x-pinctrl.dtsi 中定义：

```
/ { 
    pinctrl: pinctrl@20008000 {
        compatible = "rockchip,rk312x-pinctrl";
        ...		
        gpio0_i2c1 {
            i2c1_sda:i2c1-sda {
                rockchip,pins = <I2C1_SDA>;
                rockchip,pull = <VALUE_PULL_DEFAULT>;
            }; 
            i2c1_scl:i2c1-scl {
                rockchip,pins = <I2C1_SCL>;
                rockchip,pull = <VALUE_PULL_DEFAULT>;
            }; 
            i2c1_gpio: i2c1-gpio {
                rockchip,pins = <FUNC_TO_GPIO(I2C1_SDA)>, <FUNC_TO_GPIO(I2C1_SCL)>;
                rockchip,pull = <VALUE_PULL_DEFAULT>;
            };
        };
  }
```

I2C1_SDA, I2C1_SCL 的定义在 /kernel/arch/arm/boot/dts/include/dt-bindings/pinctrl/rockchip-rk312x.h 中：

```
#define GPIO0_A3 0x0a30#define I2C1_SDA 0x0a31#define MMC1_CMD 0x0a32 #define GPIO0_A2 0x0a20#define I2C1_SCL 0x0a21
```

FUN_TO_GPIO 的定义在 /kernel/arch/arm/boot/dts/include/dt-bindings/pinctrl/rockchip.h 中：

```
#define FUNC_TO_GPIO(m)		((m) & 0xfff0)
```

也就是说 FUNC_TO_GPIO(I2C1_SDA) == GPIO0_A3, FUNC_TO_GPIO(I2C1_SCL) == GPIO7_A2 。
像 0x0a31 这样的值是有编码规则的：

```
0 a3 1
| |  `- func
| `---- offset
`------ bank
```

0x0a31 就表示 GPIO0_A3 func1, 即 I2C1_SDA 。

在复用时，如果选择了 "default" （即 i2c 功能),系统会应用 i2c1_sda 和 i2c1_scl 这两个 pinctrl，最终得将 GPIO0_A3 和 GPIO0_A2 两个针脚切换成对应的 i2c 功能；而如果选择了 "gpio" ，系统会应用 i2c1_gpio 这个 pinctrl，将 GPIO0_A3 和 GPIO0_A2 两个针脚还原为 GPIO 功能。  
我们看看 i2c 的驱动程序 /kernel/drivers/i2c/busses/i2c-rockchip.c 是如何切换复用功能的：

```
static int rockchip_i2c_probe(struct platform_device *pdev){
    struct rockchip_i2c *i2c = NULL;
    struct resource *res;
    struct device_node *np = pdev->dev.of_node;
    int ret;
    // ...
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
    // ...
}
```

首先是调用 of_get_gpio 取出设备树中 i2c1 结点的 gpios 属于所定义的两个 gpio:

```
gpios = <&gpio0 GPIO_A3 GPIO_ACTIVE_LOW>, <&gpio0 GPIO_A2 GPIO_ACTIVE_LOW>;
```

然后是调用 devm_gpio_request 来申请 gpio，接着是调用 pinctrl_lookup_state 来查找 “gpio” 状态，而默认状态 "default" 已经由框架保存到 i2c->dev-pins->default_state 中了。   
最后调用 pinctrl_select_state 来选择是 "default" 还是 "gpio" 功能。
下面是常用的复用 API 定义：

```
#include <linux/pinctrl/consumer.h> 
struct device {
    //...
    #ifdef CONFIG_PINCTRL
    struct dev_pin_info	*pins;#endif//...}; 
    struct dev_pin_info {struct pinctrl *p;
    struct pinctrl_state *default_state;
    #ifdef CONFIG_PMstruct pinctrl_state *sleep_state;
    struct pinctrl_state *idle_state;#endif}; 
    struct pinctrl_state * pinctrl_lookup_state(struct pinctrl *p, const char *name); 
    int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
    //...
};
```