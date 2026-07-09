# GPIO 使用

## 简介

GPIO, 全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。

RK3288 有 9 组 GPIO bank： GPIO0，GPIO1, ..., GPIO8。每组又以 A0~A7, B0~B7, C0~C7, D0~D7 作为编号区分（不是所有 bank 都有全部编号，例如 GPIO5 就只有 B0~B7, C0~C3)。

每个 GPIO 口除了通用输入输出功能外，还可能有其它复用功能，例如 GPIO5_B4，可以复用成以下功能之一：

* spi0_clk
* ts0_data4
* uart4exp_ctsn

每个 GPIO 口的驱动电流、上下拉和重置后的初始状态都不尽相同，详细情况请参考[TRM 文档](http://www.t-firefly.com/doc/download/51.html#other_143)。

RK3288 的 GPIO 驱动是在以下 pinctrl 文件中实现的：

```bash
kernel/drivers/pinctrl/pinctrl-rockchip.c
```

其核心是填充 GPIO bank 的方法和参数，并调用 gpiochip_add 注册到内核中。

## 使用

开发板有两个电源 LED 灯是 GPIO 口控制的，分别是：


![](../../../rk3288_img/gpio_led_1.png)
![](../../../rk3288_img/gpio_led_2.png)


从电路图上看，GPIO 口输出低电平时灯亮，高电平时灯灭。


## 输入输出

下面以电源 LED 灯的驱动为例，讲述如何在内核编写代码控制 GPIO 口的输出。

首先需要在 dts (Device Tree) 文件 firefly-rk3288-aio-3288c.dts 中增加驱动的资源描述：


```dts
firefly-led{
    compatible = "firefly,led";
    led-work = <&gpio8 GPIO_A6 GPIO_ACTIVE_HIGH>;
    led-power = <&gpio8 GPIO_A1 GPIO_ACTIVE_LOW>;
    status = "okay";
};
```


这里定义了两颗 LED 灯的 GPIO 设置：


```dts
led-work  GPIO8_A6  GPIO_ACTIVE_HIGH
led-power GPIO8_A1  GPIO_ACTIVE_LOW
```


GPIO_ACTIVE_LOW 表示低电平有效（灯亮），如果是高电平有效，需要替换为 GPIO_ACTIVE_HIGH。

之后在驱动程序中加入对 GPIO 口的申请和控制则可：

```c
#ifdef  CONFIG_OF
#include <linux/of.h>
#include <linux/of_gpio.h>
#endif

static int firefly_led_probe(struct platform_device *pdev){
    int ret = -1;int gpio, flag;
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

调用 gpio_direction_output 就可以设置输出高还是低电平，因为是 GPIO_ACTIVE_LOW ，如果要灯亮，需要写入 0。

实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

```c
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

下面是常用的 GPIO API 定义：

```c
#include <linux/gpio.h>
#include <linux/of_gpio.h>
enum of_gpio_flags {OF_GPIO_ACTIVE_LOW = 0x1,};
int of_get_named_gpio_flags(struct device_node *np, const char *propname,int index, enum of_gpio_flags *flags);
int gpio_is_valid(int gpio);
int gpio_request(unsigned gpio, const char *label);
void gpio_free(unsigned gpio);
int gpio_direction_input(int gpio);
int gpio_direction_output(int gpio, int v)
```

## 复用

如何定义 GPIO 有哪些功能可以复用，在运行时又如何切换功能呢？以 I2C4 为例作简单的介绍。

查规格表可知，I2C4_SDA 与 I2C4_SCL 的功能定义如下：

![](../../../rk3288_img/gpio_i2c4.png)

在 `/kernel/arch/arm/boot/dts/rk3288.dtsi` 里有：

```bash
i2c4: i2c@ff160000 {
    compatible = "rockchip,rk30-i2c";
    reg = <0xff160000 0x1000>;
    interrupts = <GIC_SPI 64 IRQ_TYPE_LEVEL_HIGH>;
    #address-cells = <1>;
    #size-cells = <0>;
    pinctrl-names = "default", "gpio";
    pinctrl-0 = <&i2c4_sda &i2c4_scl>;
    pinctrl-1 = <&i2c4_gpio>;
    gpios = <&gpio7 GPIO_C1 GPIO_ACTIVE_LOW>, <&gpio7 GPIO_C2 GPIO_ACTIVE_LOW>;
    clocks = <&clk_gates6 15>;
    rockchip,check-idle = <1>;
    status = "disabled";
};
```

* 此处，跟复用控制相关的是 pinctrl- 开头的属性：
    + pinctrl-names 定义了状态名称列表： default (i2c 功能) 和 gpio 两种状态。
    + pinctrl-0 定义了状态 0 (即 default）时需要设置的 pinctrl: i2c4_sda 和 i2c4_scl
    + pinctrl-1 定义了状态 1 (即 gpio)时需要设置的 pinctrl: i2c4_gpio

这些 pinctrl 在 `/kernel/arch/arm/boot/dts/rk3288-pinctrl.dtsi` 中定义：

```dts
/ {
    pinctrl: pinctrl@ff770000 {
        compatible = "rockchip,rk3288-pinctrl";
        ...
        gpio7_i2c4 {
            i2c4_sda:i2c4-sda {
                rockchip,pins = <I2C4TP_SDA>;
                rockchip,pull = <VALUE_PULL_DISABLE>;
                rockchip,drive = <VALUE_DRV_DEFAULT>;
                //rockchip,tristate = <VALUE_TRI_DEFAULT>;
            };
            i2c4_scl:i2c4-scl {
                rockchip,pins = <I2C4TP_SCL>;
                rockchip,pull = <VALUE_PULL_DISABLE>;
                rockchip,drive = <VALUE_DRV_DEFAULT>;
                //rockchip,tristate = <VALUE_TRI_DEFAULT>;
            };
            i2c4_gpio: i2c4-gpio {
                rockchip,pins = <FUNC_TO_GPIO(I2C4TP_SDA)>, <FUNC_TO_GPIO(I2C4TP_SCL)>;
                rockchip,drive = <VALUE_DRV_DEFAULT>;
            };
        };
        ...
    }
  }
```

I2C4TP_SDA, I2C4TP_SCL 的定义在 `/kernel/arch/arm/boot/dts/include/dt-bindings/pinctrl/rockchip-rk3288.h` 中：

```c
#define GPIO7_C1 0x7c10
#define I2C4TP_SDA 0x7c11
#define GPIO7_C2 0x7c20
#define I2C4TP_SCL 0x7c21
```

FUN_TO_GPIO 的定义在 `/kernel/arch/arm/boot/dts/include/dt-bindings/pinctrl/rockchip.h` 中：

```c
#define FUNC_TO_GPIO(m)     ((m) & 0xfff0)
```

也就是说 FUNC_TO_GPIO(I2C4TP_SDA) == GPIO7_C1, FUNC_TO_GPIO(I2C4TP_SCL) == GPIO7_C2。

像 0x7c11 这样的值是有编码规则的：

```c
7 c1 1
| |  `- func
| `---- offset
`------ bank
0x7c11 就表示 GPIO7_C1 func1, 即 i2c4tp_sda 。
```

在复用时，如果选择了 "default" （即 i2c 功能),系统会应用 i2c4_sda 和 i2c4_scl 这两个 pinctrl，最终得将 GPIO7_C1 和 GPIO7_C2 两个针脚切换成对应的 i2c 功能；而如果选择了 "gpio" ，系统会应用 i2c4_gpio 这个 pinctrl，将 GPIO7_C1 和 GPIO7_C2 两个针脚还原为 GPIO 功能。

我们看看 i2c 的驱动程序 `/kernel/drivers/i2c/busses/i2c-rockchip.c` 是如何切换复用功能的：

```c
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

首先是调用 of_get_gpio 取出设备树中 i2c4 结点的 gpios 属于所定义的两个 gpio:

```dts
gpios = <&gpio7 GPIO_C1 GPIO_ACTIVE_LOW>, <&gpio7 GPIO_C2 GPIO_ACTIVE_LOW>;
```

然后是调用 devm_gpio_request 来申请 gpio，接着是调用 pinctrl_lookup_state 来查找 “gpio” 状态，而默认状态 "default" 已经由框架保存到 i2c->dev-pins->default_state 中了。最后调用 pinctrl_select_state 来选择是 "default" 还是 "gpio" 功能。

下面是常用的复用 API 定义：

```c
#include <linux/pinctrl/consumer.h>
struct device {
    //...
    #ifdef
    CONFIG_PINCTRLstruct dev_pin_info    *pins;
    #endif
    //...
};

struct dev_pin_info {
    struct pinctrl *p;
    struct pinctrl_state *default_state;
    #ifdef CONFIG_PMstruct pinctrl_state *sleep_state;
    struct pinctrl_state *idle_state;
    #endif
};

struct pinctrl_state * pinctrl_lookup_state(struct pinctrl *p, const char *name);
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
```