# GPIO 使用

## 简介

GPIO, 全称 General-Purpose Input/Output（通用输入输出），是一种软件运行期间能够动态配置和控制的通用引脚。 所有的 GPIO 在上电后的初始状态都是输入模式，可以通过软件设为上拉或下拉，也可以设置为中断脚，驱动强度都是可编程的,其核心是填充 GPIO bank 的方法和参数，并调用 gpiochip_add 注册到内核中。


本文以 GPIO0_B4 和 GPIO4_D5 这两个 GPIO 口为例写了一份简单操作 GPIO 口的驱动，在 SDK 的路径为：`kernel/drivers/gpio/gpio-firefly.c`，以下就以该驱动为例介绍 GPIO 的操作。

板子可能并没有真的引出 GPIO0_B4 和 GPIO4_D5，这里只是举例。

## GPIO 引脚计算
iCore-3562JQ 有 5 组 GPIO bank：GPIO0~GPIO4，每组又以 A0~A7, B0~B7, C0~C7, D0~D7 作为编号区分,常用以下公式计算引脚：

GPIO 小组编号计算公式：number = group * 8 + X

GPIO pin 编号计算公式：pin = bank * 32 + number

下面演示GPIO4_D5 pin 脚计算方法：

bank = 4;   //GPIO<font color=red>4</font>_D5 => 4

group = 3;  //GPIO4_<font color=red>D</font>5 => 3  {(A=0), (B=1), (C=2), (D=3)} 

X = 5;      //GPIO4_D<font color=red>5</font> => 5

number = group * 8 + X = 3 * 8 + 5 = 29

pin = bank*32 + number=  4 * 32 + 29 = 157;

GPIO4_D5 对应的设备树属性描述为`<&gpio4 29 IRQ_TYPE_EDGE_RISING>`，由`kernel/include/dt-bindings/pinctrl/rockchip.h`的宏定义可知，也可以将 GPIO4_D5 描述为`<&gpio4 RK_PD5 IRQ_TYPE_EDGE_RISING>`。

## 用户空间使用 GPIO

当引脚没有被其它外设复用时, 我们可以通过export导出该引脚去使用。
```shell
# 将 gpio 编号写入 export
echo 157 > /sys/class/gpio/export

# 正常会生成一个 gpio+编号 文件夹
ls /sys/class/gpio/
export   gpiochip0    gpiochip255  gpiochip500  gpiochip96
gpio157  gpiochip128  gpiochip32   gpiochip64   unexport

# 查看该文件内属性
ls /sys/class/gpio/gpio157
active_low  device  direction  edge  power  subsystem  uevent  value

# 这个节点用于设置方向
cat /sys/class/gpio/gpio157/direction
in
echo out > /sys/class/gpio/gpio157/direction

# 这个节点用于设置电平值
cat /sys/class/gpio/gpio157/value
0
echo 1 > /sys/class/gpio/gpio157/value
```

## 内核空间使用

首先在 DTS 文件中增加驱动的资源描述：

```
gpio_demo: gpio_demo {
    status = "okay";
    compatible = "firefly,rk356x-gpio";
    firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
    firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
};
```

这里定义了一个脚作为一般的输出输入口：
```
firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;
```

`GPIO_ACTIVE_HIGH` 表示高电平有效，如果想要低电平有效，可以改为：`GPIO_ACTIVE_LOW`，这个属性将被驱动所读取。

然后在 probe 函数中对 DTS 所添加的资源进行解析，代码如下：

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

`of_get_named_gpio_flags` 从设备树中读取 `firefly-gpio` 和 `firefly-irq-gpio` 的 GPIO 配置编号和标志，`gpio_is_valid` 判断该 GPIO 编号是否有效，`gpio_request` 则申请占用该 GPIO。如果初始化过程出错，需要调用 `gpio_free` 来释放之前申请过且成功的 GPIO 。在驱动中调用 `gpio_direction_output` 就可以设置输出高还是低电平，这里默认输出从 DTS 获取得到的有效电平 `GPIO_ACTIVE_HIGH`，即为高电平，如果驱动正常工作，可以用万用表测得对应的引脚应该为高电平。实际中如果要读出 GPIO，需要先设置成输入模式，然后再读取值：

```c
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```

下面是常用的 GPIO API 定义：

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

## 中断

在 Firefly 的例子程序中还包含了一个中断引脚，GPIO 口的中断使用与 GPIO 的输入输出类似，首先在 DTS 文件中增加驱动的资源描述：

```
firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
```

`IRQ_TYPE_EDGE_RISING` 表示中断由上升沿触发，当该引脚接收到上升沿信号时可以触发中断函数。 这里还可以配置成如下：

```c
IRQ_TYPE_NONE         //默认值，无定义中断触发类型
IRQ_TYPE_EDGE_RISING  //上升沿触发
IRQ_TYPE_EDGE_FALLING //下降沿触发
IRQ_TYPE_EDGE_BOTH    //上升沿和下降沿都触发
IRQ_TYPE_LEVEL_HIGH   //高电平触发
IRQ_TYPE_LEVEL_LOW    //低电平触发
```

然后在 probe 函数中对 DTS 所添加的资源进行解析，再做中断的注册申请，代码如下：

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

调用 `gpio_to_irq` 把 GPIO 的 PIN 值转换为相应的 IRQ 值，调用 `gpio_request` 申请占用该 IO 口，调用 `request_irq` 申请中断，如果失败要调用 `free_irq` 释放，该函数中 `gpio_info-firefly_irq` 是要申请的硬件中断号，`firefly_gpio_irq` 是中断函数，`gpio_info->firefly_irq_mode` 是中断处理的属性，`firefly-gpio` 是设备驱动程序名称，`gpio_info` 是该设备的 `device` 结构，在注册共享中断时会用到。

## 复用

GPIO 口除了通用输入输出、中断功能外，还可能有其它复用功能，如 GPIO0_B4, 除了 GPIO 的功能，它还能作为 I2C1_SDA_M0

那么在使用作 GPIO 口时，就需要注意是否被复用为其他功能了。

例如使用 GPIO0_B4 作 GPIO 功能时就需要在设备树中将 I2C1 disabled 掉。
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

复用的设置是通过 pinctrl 框架来实现的。在 i2c1 的设备树节点中有这样两行属性：
```
pinctrl-names = "default";
pinctrl-0 = <&i2c1m0_xfer>;
```
这就是设置了 pinctrl，其中 i2c1m0_xfer 定义在 rk3562-pinctrl.dtsi 中：
```
i2c1 {
    /omit-if-no-ref/
    i2c1m0_xfer: i2c1m0-xfer {
        rockchip,pins =
            /* i2c1_scl_m0 */
            <0 RK_PB3 1 &pcfg_pull_none_smt>,
            /* i2c1_sda_m0 */
            <0 RK_PB4 1 &pcfg_pull_none_smt>; // 这里就设置了 GPIO0_B4 为 func1，即 i2c1_sda_m0 功能
    };
	...
};
```
所以如果没有 disabled 掉 i2c1 的话，i2c1 中的 pinctrl 就会生效，将 GPIO0_B4 配置成了 i2c 的功能。

更多内容请查看文档 SDK/docs/cn/Common/PINCTRL/Rockchip_Developer_Guide_Linux_Pinctrl_CN.pdf

## 调试方法

### IO 指令

GPIO 调试有一个很好用的工具，那就是 IO 指令，iCore-3562JQ 的系统默认已经内置了 IO 指令，使用 IO 指令可以实时读取或写入每个 IO 口的状态，这里简单介绍 IO 指令的使用。首先查看 IO 指令的帮助：

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

从帮助上可以看出，如果要读或者写一个寄存器，可以用：

```
io -4 -r 0x1000 //读从0x1000起的4位寄存器的值
io -4 -w 0x1000 //写从0x1000起的4位寄存器的值
```

使用示例：

* 查看 GPIO1_B3 引脚的复用情况
* 从主控的 datasheet 查到 GPIO1 对应寄存器基地址为：0xff320000
* 从主控的 datasheet 查到 GPIO1B_IOMUX 的偏移量为：0x00014
* GPIO1_B3 的 iomux 寄存器地址为：基址(Operational Base) + 偏移量(offset)=0xff320000+0x00014=0xff320014
* 用以下指令查看GPIO1_B3的复用情况：

```
# io -4 -r 0xff320014
ff320014:  0000816a
```

*  从datasheet查到[7:6]：

```
gpio1b3_sel
GPIO1B[3] iomux select
2'b00: gpio
2'b01: i2c4sensor_sda
2'b10: reserved
2'b11: reserved
```

因此可以确定该 GPIO 被复用为 i2c4sensor_sda。

*  如果想复用为 GPIO，可以使用以下指令设置：

```
# io -4 -w 0xff320014 0x0000812a
```

### Debugfs

Debugfs 文件系统目的是为开发人员提供更多内核数据，方便调试。 这里 GPIO 的调试也可以用 Debugfs 文件系统，获得更多的内核信息。GPIO 在 Debugfs 文件系统中的接口为 `/sys/kernel/debug/gpio`。

## FAQs

### Q1: 如何将 PIN 的 MUX 值切换为一般的 GPIO？

A1: 当使用 GPIO request 时候，会将该 PIN 的 MUX 值强制切换为 GPIO，所以使用该 PIN 脚为 GPIO 功能的时候确保该 PIN 脚没有被其他模块所使用。

### Q2: 为什么我用 IO 指令读出来的值都是 0x00000000？

A2: 如果用 IO 命令读某个 GPIO 的寄存器，读出来的值异常,如 0x00000000 或 0xffffffff 等，请确认该 GPIO 的 CLK 是不是被关了，GPIO 的 CLK 是由 CRU 控制，可以通过读取 datasheet 下面 CRU_CLKGATE_CON* 寄存器来查到 CLK 是否开启，如果没有开启可以用 io 命令设置对应的寄存器，从而打开对应的 CLK，打开 CLK 之后应该就可以读到正确的寄存器值了。

### Q3: 测量到 PIN 脚的电压不对应该怎么查？

A3: 测量该 PIN 脚的电压不对时，如果排除了外部因素，可以确认下该 PIN 所在的 IO 电压源是否正确。
