# 定时器使用

## 前言

RK3399有 12 个 Timers (timer0-timer11)，有 12 个 Secure Timers(stimer0~stimer11) 和 2 个 Timers(pmutimer0~pmutimer1)， 我们主要用到的是 Timers(timer0-timer11) 时钟频率为 24MHZ ，工作模式有 `free-running` 和 `user-defined count` 模式。

## 框架图

![](../../../rk3399_img/include/RK3399/timer_frame.jpg)

## 工作模式

* `user-defined count`：Timer 先载入初始值到 TIMERn_LOAD_COUNT3 和 TIMER_LOADn_COUNT2 寄存器， 当时间累加的值在寄存器 TIMERn_LOAD_COUNT1 和 TIMERn_LOAD_COUNT0 时，将不会自动载入到计数寄存器。 用户需要重新关闭计数器和然后重新设置计数器相关才能继续工作。
* `free-running`：Timer 先载入初始值到 TIMER_LOAD_COUNT3 和 TIMER_LOAD_COUNT2 寄存器， 当时间累加的值在寄存器 TIMERn_LOAD_COUNT1 和 TIMERn_LOAD_COUNT0 时，Timer 将一直自动加载计数寄存器。

## 软件配置

1.在 dts 文件中定义 Timer 的相关配置 `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi`

```
rktimer: rktimer@ff850000 {
	compatible = "rockchip,rk3399-timer";
	reg = <0x0 0xff850000 0x0 0x1000>;
	interrupts = <GIC_SPI 81 IRQ_TYPE_LEVEL_HIGH 0>;
	clocks = <&cru PCLK_TIMER0>, <&cru SCLK_TIMER00>;
	clock-names = "pclk", "timer";
};
```

其中定义的 Timer0 的寄存器和中断号和时钟等。

其他 Timer 对应的中断号可看如下图片：

![](../../../rk3399_img/include/RK3399/timer_interrupt.jpg)

2.对应的驱动文件 `Kernel/drivers/clocksource/rockchip_timer.c`

## 对应寄存器和使用

1. 寄存器如下图片：

![](../../../rk3399_img/include/RK3399/timer_register.jpg)

2. 使用 `io` 命令方式查看对应寄存器：

```
root@host_name:/ # io -4 0xff85001c  //查看当前控制寄存器的状态
ff85001c:  00000007

root@host_name:/ # io -4 0xff850000  //查看寄存器时时的值
ff850000:  0001639f
```

控制对应寄存器：

```
root@host_name:/ # io -4 -w 0xff85001c 0x06  //关闭时间计数功能
```
