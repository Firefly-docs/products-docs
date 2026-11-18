# GPIO 使用

## 简介 [step]

GPIO，全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。所有的 GPIO 在上电后的初始状态都是输入模式，可以通过软件设为上拉或下拉，也可以设置为中断脚，驱动强度都是可编程的。

Firefly-R3572A 通过 J1 双排排针引出 GPIO 信号（`GPIO1_C7_d`，上拉 10K），可配置为输入或输出使用。Linux 下推荐使用 libgpiod 工具操作 GPIO。

## 硬件连接 [step]

| 排针序号 | 信号 | 电平 |
| --- | --- | --- |
| 20 | GPIO1_C7_d | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="注意" type="warn">

GPIO 电平为 1.8 V，外接电路不要直接接入 3.3 V 或 5 V 信号，否则可能损坏芯片。

</Callout>

## GPIO 编号计算 [step]

RK3572 的 GPIO 编号按 `组号 × 32 + 组内字母 × 8 + 序号` 计算：

* 组号：`GPIO1_C7_d` 的 `1`
* 组内字母：A=0、B=1、C=2、D=3，`C` 为 2
* 序号：`7`

因此 `GPIO1_C7` 对应系统编号：

```text
1 × 32 + 2 × 8 + 7 = 55
```

`GPIO1_C7` 对应的设备树属性描述为 `<&gpio1 55 GPIO_ACTIVE_HIGH>`，由 `kernel-6.12/include/dt-bindings/pinctrl/rockchip.h` 的宏定义可知，也可以写作 `<&gpio1 RK_PC7 GPIO_ACTIVE_HIGH>`。

当 GPIO1_C7 没有被其它外设复用时，也可以通过 sysfs 接口导出该引脚使用：

```shell
:/ # echo 55 > /sys/class/gpio/export
:/ # ls /sys/class/gpio/gpio55
active_low  device  direction  edge  power  subsystem  uevent  value
:/ # cat /sys/class/gpio/gpio55/direction
in
```

## 输出控制 [step]

将 GPIO 配置为输出并设置电平（libgpiod）：

```bash
gpioset gpiochip1 7=1    # 输出高电平
gpioset gpiochip1 7=0    # 输出低电平
```

## 输入读取 [step]

将 GPIO 配置为输入并读取电平：

```bash
gpioget gpiochip1 7
```

## 内核驱动使用 [step]

在 DTS 文件中增加驱动的资源描述：

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

`GPIO_ACTIVE_HIGH` 表示高电平有效，如果想要低电平有效，可以改为 `GPIO_ACTIVE_LOW`，这个属性将被驱动所读取。

内核驱动中通过 GPIO 描述符接口申请引脚：

```c
#include <linux/gpio/consumer.h>

struct gpio_desc *gpiod;
gpiod = gpiod_get(&pdev->dev, "demo", GPIOD_OUT_LOW);
gpiod_set_value(gpiod, 1);
gpiod_put(gpiod);
```

也可以使用传统的 GPIO 数值接口，从设备树中读取 GPIO 配置编号和标志：

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

实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

```c
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

## 中断 [step]

GPIO 口的中断使用与 GPIO 的输入输出类似，在 DTS 文件中增加驱动的中断资源描述：

```text
kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-demo.dtsi
gpio {
	compatible = "firefly-gpio";
	firefly-irq-gpio = <&gpio1 RK_PC7 IRQ_TYPE_EDGE_RISING>;  /* GPIO1_C7_d */
};
```

`IRQ_TYPE_EDGE_RISING` 表示中断由上升沿触发，当该引脚接收到上升沿信号时可以触发中断函数。还可以配置成如下触发方式：

```text
IRQ_TYPE_NONE         //默认值，无定义中断触发类型
IRQ_TYPE_EDGE_RISING  //上升沿触发
IRQ_TYPE_EDGE_FALLING //下降沿触发
IRQ_TYPE_EDGE_BOTH    //上升沿和下降沿都触发
IRQ_TYPE_LEVEL_HIGH   //高电平触发
IRQ_TYPE_LEVEL_LOW    //低电平触发
```

然后在 probe 函数中对 DTS 所添加的资源进行解析，再做中断的注册申请：

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

static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) //中断函数
{
	printk("Enter firefly gpio irq test program!\n");
	return IRQ_HANDLED;
}
```

调用 `gpio_to_irq` 把 GPIO 的 PIN 值转换为相应的 IRQ 值，调用 `gpio_request` 申请占用该 IO 口，调用 `request_irq` 申请中断，如果失败要调用 `free_irq` 释放。该函数中 `firefly_irq` 是要申请的硬件中断号，`firefly_gpio_irq` 是中断函数，`firefly_irq_mode` 是中断处理的属性，`firefly-gpio` 是设备驱动程序名称，`gpio_info` 是该设备的 `device` 结构，在注册共享中断时会用到。

## 复用 [step]

`该案例仅供参考，最终以实际硬件接口为准`

GPIO 口除了通用输入输出、中断功能外，还可能有其它复用功能。以 J1 排针引出的 `I2C2` 为例，其引脚同时具有 GPIO 与 I2C 功能，查看 `/d/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins` 可以确认各引脚当前的作用。如果引脚已被复用为 I2C，需要在 DTS 中关闭对应节点后再作为 GPIO 使用：

```text
&i2c2 {
    status = "disabled";
};
```

在运行时切换引脚功能依赖内核 pinctrl 框架：控制器节点的 `pinctrl-names` 定义了状态名称列表（如 default 功能状态和 gpio 状态），`pinctrl-0`、`pinctrl-1` 分别定义各状态使用的引脚复用配置。驱动的 probe 流程会通过 `pinctrl_select_state` 选择 DTS 中设置的 pinctrl 状态。详细介绍可以参考 `SDK/docs` 下的 `Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf`。

## 调试方法 [step]

### GPIO 调试接口

Debugfs 文件系统目的是为开发人员提供更多内核数据，方便调试。GPIO 在 Debugfs 文件系统中的接口为 `/sys/kernel/debug/gpio`：

```shell
:/ # cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-31, parent: platform/2002c000.gpio, gpio0:
 ...
gpiochip1: GPIOs 32-63, parent: platform/20030000.gpio, gpio1:
 gpio-55  (                    |firefly-gpio        ) out hi
 ...
```

（以上输出为参考示例，实际内容以运行的系统为准。）从读取到的信息中可以知道内核把 GPIO 当前的状态都列出来了，例如 gpio-55（GPIO1_C7）输出高电平（out hi）。

### 查看 pinmux-pins

```shell
:/ # cat /d/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins
```

得到结果（参考示例）：

```text
Pinmux settings per pin
Format: pin (name): mux_owner gpio_owner hog?
pin 0 (gpio0-0): wireless-bluetooth gpio0:0 function wireless-bluetooth group bt-irq-gpio
pin 1 (gpio0-1): (MUX UNCLAIMED) (GPIO UNCLAIMED)
pin 2 (gpio0-2): (MUX UNCLAIMED) (GPIO UNCLAIMED)
...
```

解析：`pin 0` 这一列表示引脚编号，`gpio0-0` 这一列表示 gpio 组编号，`MUX UNCLAIMED` 这一列表示数据选择器的拥有者，`GPIO UNCLAIMED` 这一列表示 gpio 的拥有者。

其中 `MUX UNCLAIMED` 表示该引脚还没有被节点使用 pinctrl 去进行控制。例如节点 i2c2 被启用后，它拥有 pinctrl-0 属性，对引脚功能作出修改复用为 i2c，则该引脚的信息会变为被对应 i2c 节点使用的状态；用 gpio_demo 例子注册该引脚后，引脚信息会变为被名为 gpio_demo 的节点使用 pinctrl 配置，并被申请为 gpio。

## FAQs [step]

### Q1: 如何将 PIN 的 MUX 值切换为一般的 GPIO？

A1: 当使用 GPIO request 时候，会将该 PIN 的 MUX 值强制切换为 GPIO，所以使用该 PIN 脚为 GPIO 功能的时候确保该 PIN 脚没有被其他模块所使用。

### Q2: 为什么我用 IO 指令读出来的值都是 0x00000000？

A2: 如果用 IO 命令读某个 GPIO 的寄存器，读出来的值异常，如 0x00000000 或 0xffffffff 等，请确认该 GPIO 的 CLK 是不是被关了，GPIO 的 CLK 是由 CRU 控制，可以通过读取 datasheet 下面 CRU_CLKGATE_CON* 寄存器来查到 CLK 是否开启，如果没有开启可以用 io 命令设置对应的寄存器，从而打开对应的 CLK，打开 CLK 之后应该就可以读到正确的寄存器值了。

### Q3: 测量到 PIN 脚的电压不对应该怎么查？

A3: 测量该 PIN 脚的电压不对时，如果排除了外部因素，可以确认下该 PIN 所在的 IO 电压源是否正确，以及 IO-Domain 配置是否正确。

### Q4: gpio_set_value() 与 gpio_direction_output() 有什么区别？

A4: 如果使用该 GPIO 时，不会动态的切换输入输出，建议在开始时就设置好 GPIO 输出方向，后面拉高拉低时使用 gpio_set_value() 接口，而不建议使用 gpio_direction_output()，因为 gpio_direction_output 接口里面有 mutex 锁，对中断上下文调用会有错误异常，且相比 gpio_set_value，gpio_direction_output 所做事情更多，浪费。
