# GPIO 使用

## 简介

GPIO，全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。 所有的 GPIO 在上电后的初始状态都是输入模式，可以通过软件设为上拉或下拉，也可以设置为中断脚，驱动强度都是可编程的，其核心是填充 GPIO bank 的方法和参数，并调用 gpiochip_add 注册到内核中。




## GPIO引脚计算
Core-1126-JD4 有 5 组 GPIO bank：GPIO0~GPIO4，每组又以 A0~A7, B0~B7, C0~C7, D0~D7 作为编号区分，常用以下公式计算引脚：

```shell
GPIO pin脚计算公式：pin = bank * 32 + number

GPIO 小组编号计算公式：number = group * 8 + X
```
下面演示GPIO2_C0 pin脚计算方法：

bank = 2;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO<font color=red>2</font>_C0 => 2,  bank ∈ [0,4]

group = 2;  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO2_<font color=red>C</font>0 => 2,  group ∈ {(A=0), (B=1), (C=2), (D=3)}

X = 0;    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//GPIO2_C<font color=red>0</font> => 0, X ∈ [0,7]

number = group * 8 + X = 2 * 8 + 0 = 16

pin = bank*32 + number= 2 * 32 + 16 = 80;

GPIO2_C0 对应的设备树属性描述为:<&gpio2 16 GPIO_ACTIVE_HIGH>,由`kernel/include/dt-bindings/pinctrl/rockchip.h`的宏定义可知，也可以将GPIO2_C0描述为<&gpio2 RK_PC0 GPIO_ACTIVE_HIGH>。

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
...
```
当 GPIO2_C0 脚没有被其它外设复用时，我们可以通过 export 导出该引脚去使用

```shell
[root@RV1126_RV1109:/]# ls /sys/class/gpio/
export  gpiochip0  gpiochip128  gpiochip32  gpiochip503  gpiochip511  gpiochip64  gpiochip96  unexport
[root@RV1126_RV1109:/]# echo 80 > /sys/class/gpio/export
[root@RV1126_RV1109:/]# ls /sys/class/gpio/
export  gpio80  gpiochip0  gpiochip128  gpiochip32  gpiochip503  gpiochip511  gpiochip64  gpiochip96  unexport
[root@RV1126_RV1109:/]# ls /sys/class/gpio/gpio80
active_low  device  direction  edge  power  subsystem  uevent  value
```


## 输入输出

### 应用层设置

在上述 export 导出引脚后，可以直接设置 gpio 为输入模式或者输出模式

* 查看当前 gpio 的模式
```shell
[root@RV1126_RV1109:/]# cat /sys/class/gpio/gpio80/direction
in
```

* 设置为输入模式

```shell
[root@RV1126_RV1109:/]# echo in > /sys/class/gpio/gpio80/direction
```

* 在输入模式下，直接查看 value 文件的值即可知道输入的电平

```shell
[root@RV1126_RV1109:/]# cat /sys/class/gpio/gpio80/value
0
```

* 设置为输出模式

```shell
[root@RV1126_RV1109:/]# echo out > /sys/class/gpio/gpio80/direction
```

* 设置输出高电平

```shell
[root@RV1126_RV1109:/]# echo 1 > /sys/class/gpio/gpio80/value
```

* 设置输出低电平

```shell
[root@RV1126_RV1109:/]# echo 0 > /sys/class/gpio/gpio80/value
```



### 驱动层设置

`of_get_named_gpio_flags` 从设备树中读取 `gpio` 和 `irq-gpio` 的 GPIO 配置编号和标志，`gpio_is_valid` 判断该 GPIO 编号是否有效，`gpio_request` 则申请占用该 GPIO。如果初始化过程出错，需要调用 `gpio_free` 来释放之前申请过且成功的 GPIO 。在驱动中调用 `gpio_direction_output` 就可以设置输出高还是低电平，这里默认输出从 DTS 获取得到的有效电平 `GPIO_ACTIVE_HIGH`，即为高电平，如果驱动正常工作，可以用万用表测得对应的引脚应该为高电平。实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

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

IRQ_TYPE_EDGE_RISING 表示中断由上升沿触发，当该引脚接收到上升沿信号时可以触发中断函数。 这里还可以配置成如下：

```
IRQ_TYPE_NONE         //默认值，无定义中断触发类型
IRQ_TYPE_EDGE_RISING  //上升沿触发
IRQ_TYPE_EDGE_FALLING //下降沿触发
IRQ_TYPE_EDGE_BOTH    //上升沿和下降沿都触发
IRQ_TYPE_LEVEL_HIGH   //高电平触发
IRQ_TYPE_LEVEL_LOW    //低电平触发
```

然后在 probe 函数中对 DTS 所添加的资源进行解析，再做中断的注册申请，例如代码如下：

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

static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) //中断函数
{
	printk("Enter firefly gpio irq test program!\n");
	return IRQ_HANDLED;
}
```

调用 `gpio_to_irq` 把 GPIO 的 PIN 值转换为相应的 IRQ 值，调用 `gpio_request` 申请占用该 IO 口，调用 `request_irq` 申请中断，如果失败要调用 `free_irq` 释放，该函数中 `gpio_info-firefly_irq` 是要申请的硬件中断号，`firefly_gpio_irq` 是中断函数，`gpio_info->firefly_irq_mode` 是中断处理的属性，`firefly-gpio` 是设备驱动程序名称，`gpio_info` 是该设备的 `device` 结构，在注册共享中断时会用到。

## 复用
`该案例仅供参考，最终以实际硬件接口为准`<br>
GPIO 口除了通用输入输出、中断功能外，还可能有其它复用功能，以GPIO0_C3为例，就有如下几个功能：

|  func0  |  func1  |  func2  |
| --- | --- | --- |
| GPIO0_C3 | I2C2_SDA | PWM5_M0 |

查看 `/sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins`，查看各个引脚的作用，如果发现GPIO0_C2和GPIO0_C3被复用为I2c，则需要在dts中关闭它

```
&i2c2 {
    status = "disabled";
};
```
**Note:** 此处 **GPIO0_C2** 仅作示例，实际使用中不推荐如此修改

上面介绍了在DTS上修改，那在运行时又如何切换功能呢？下面以 I2C2 为例作简单的介绍，详细介绍可以参考`docs/Common/Pin-Ctrl/Rockchip-Developer-Guide-Linux-Pin-Ctrl-CN.pdf`。

查规格表可知，I2C2_SDA 与 I2C2_SCL 的功能定义如下：

|  Pad#  |  func0  |  func1  |  func2  |
| --- | --- | --- | --- |
| PWM4_M0 | GPIO0_C2 | I2C2_SCL | PWM4_M0 |
| PWM5_M0 | GPIO0_C3 | I2C2_SDA | PWM4_M0 |

在 `kernel/arch/arm/boot/dts/rv1126.dtsi` 里有：

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

跟复用控制相关的是 `pinctrl-` 开头的属性：

* pinctrl-names 定义了状态名称列表： default (i2c 功能) 和 gpio 两种状态。
* pinctrl-0 定义了状态 0 (即 default）时需要设置的 pinctrl: &i2c2_xfer
* pinctrl-1 定义了状态 1 (即 gpio)时需要设置的 pinctrl: &i2c2_gpio

这些 pinctrl 在 `kernel/arch/arm/boot/dts/rv1126.dtsi` 中这样定义：

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

在`kernel/arch/arm/boot/dts/rv1126.dtsi`中有i2c2的定义

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

RK_FUNC_GPIO 的定义在 `kernel/include/dt-bindings/pinctrl/rockchip.h` ，此处简写作0：

```
#define RK_FUNC_GPIO	0
```

知道了上面关于i2c2的定义后，在 kernel 的 dts 文件中为i2c2节点添加gpio的资源

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

i2c驱动注册流程如下：
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
`pinctrl_select_state`是选择pinctrl的函数,它用来选择我们dts中设置的pinctrl。

## 调试方法

### IO指令

GPIO 调试有一个很好用的工具，那就是 IO 指令，Core-1126-JD4 的 Linux 系统默认已经内置了 IO 指令，使用 IO 指令可以实时读取或写入每个 IO 口的状态，这里简单介绍 IO 指令的使用。首先查看 IO 指令的帮助：

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

从帮助上可以看出，如果要读或者写一个寄存器，可以用：

```
io -4 -r 0x1000 //读从0x1000起的4位寄存器的值
io -4 -w 0x1000 //写从0x1000起的4位寄存器的值
```

使用示例：

* 查看GPIO0_C2引脚的复用情况
* 从主控的datasheet查到GPIO0对应寄存器基地址为：0xFE020000
* 从主控的datasheet查到GPIO0D_IOMUX的偏移量为：0x0010
* GPIO0_C2的iomux寄存器地址为：基址(Operational Base) + 偏移量(offset)=0xFE020000+0x0010=0xFE020010
* 用以下指令查看GPIO1_D0的复用情况：

```
[root@RV1126_RV1109:/]# io -4 -r 0xFE020010
fe020010:  00000020
```

*  从 trm datasheet 查到[10:8]：

```
gpio0c2_sel
3'h0: GPIO0_C2_d
3'h1: I2C2_SCL
3'h3: PWM4_PIN_M0
Others: Reserved
```

因此可以确定该 GPIO 被复用为 GPIO0_C2_d。

*  如果想复用为 I2C2_SCL，可以使用以下指令设置：

```
[root@RV1126_RV1109:/]# io -4 -w 0xFE020010 0x1000120
```

### GPIO 调试接口

Debugfs 文件系统目的是为开发人员提供更多内核数据，方便调试。 这里 GPIO 的调试也可以用 Debugfs 文件系统，获得更多的内核信息。GPIO 在 Debugfs 文件系统中的接口为 `/sys/kernel/debug/gpio`，可以这样读取该接口的信息：

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

从读取到的信息中可以知道，内核把 GPIO 当前的状态都列出来了，以 GPIO1 组为例，gpio-55(GPIO1_C7) 输出低电平 (out lo)。

### 查看 pinmux-pins

使用命令 
```
[root@RV1126_RV1109:/]# cat /sys/kernel/debug/pinctrl/pinctrl-rockchip-pinctrl/pinmux-pins
```

得到结果
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
解析：
`pin 0`这一列表示引脚编号，`gpio0-0`这一列表示gpio组编号，后面`MUX UNCLAIMED`这一列表示数据选择器的拥有者，`GPIO UNCLAIMED`这一列表示gpio的拥有者。

其中 `MUX UNCLAIMED` 表示该引脚还没有被节点使用pinctrl去进行控制。`GPIO UNCLAIMED`表示还没有注册的gpio使用该引脚

## FAQs

### Q1: 如何将 PIN 的 MUX 值切换为一般的 GPIO？

A1: 当使用 GPIO request 时候，会将该 PIN 的 MUX 值强制切换为 GPIO，所以使用该 PIN 脚为 GPIO 功能的时候确保该 PIN 脚没有被其他模块所使用。

### Q2: 为什么我用 IO 指令读出来的值都是 0x00000000？

A2: 如果用 IO 命令读某个 GPIO 的寄存器，读出来的值异常,如 0x00000000 或 0xffffffff 等，请确认该 GPIO 的 CLK 是不是被关了，GPIO 的 CLK 是由 CRU 控制，可以通过读取 datasheet 下面 CRU_CLKGATE_CON* 寄存器来查到 CLK 是否开启，如果没有开启可以用 io 命令设置对应的寄存器，从而打开对应的 CLK，打开 CLK 之后应该就可以读到正确的寄存器值了。

### Q3: 测量到 PIN 脚的电压不对应该怎么查？

A3: 测量该 PIN 脚的电压不对时，如果排除了外部因素，可以确认下该 PIN 所在的 IO 电压源是否正确，以及 IO-Domain 配置是否正确。

### Q4: gpio_set_value() 与 gpio_direction_output() 有什么区别？

A4: 如果使用该 GPIO 时，不会动态的切换输入输出，建议在开始时就设置好 GPIO 输出方向，后面拉高拉低时使用 gpio_set_value() 接口，而不建议使用 gpio_direction_output(), 因为 gpio_direction_output 接口里面有 mutex 锁，对中断上下文调用会有错误异常，且相比 gpio_set_value，gpio_direction_output 所做事情更多，浪费。
