
# TIMER

## Introduction

RK3399 has 12 Timers (timer0-timer11), 12 Secure Timers (stimer0~stimer11) and 2 Timers (pmutimer0~pmutimer1), we mainly use Timers (timer0-timer11) clock frequency is 24MHZ, working mode There are `free-running` and `user-defined count` modes.

## The framework

![](../../../rk3399_img/timer_frame.jpg)

## Working mode

* `user-defined count`: Timer first loads the initial value into the TIMERn_LOAD_COUNT3 and TIMER_LOADn_COUNT2 registers. When the time accumulated value is in the registers TIMERn_LOAD_COUNT1 and TIMERn_LOAD_COUNT0, it will not be automatically loaded into the count register. The user needs to turn off the counter and then reset the counter to continue working.
* `free-running`: Timer first loads the initial value into the TIMER_LOAD_COUNT3 and TIMER_LOAD_COUNT2 registers. When the time accumulated value is in the registers TIMERn_LOAD_COUNT1 and TIMERn_LOAD_COUNT0, the Timer will always load the count register automatically.

## Software configuration

1. Define the relevant configuration of Timer in the dts file `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi`

```
rktimer: rktimer@ff850000 {
    compatible = "rockchip,rk3399-timer";
    reg = <0x0 0xff850000 0x0 0x1000>;
    interrupts = <GIC_SPI 81 IRQ_TYPE_LEVEL_HIGH 0>;
    clocks = <&cru PCLK_TIMER0>, <&cru SCLK_TIMER00>;
    clock-names = "pclk", "timer";
};
```

The registers of Timer0, interrupt number and clock defined therein.

The interrupt numbers corresponding to other Timers can be seen in the following pictures:

![](../../../rk3399_img/include/RK3399/timer_interrupt.jpg)

2. Corresponding driver file `Kernel/drivers/clocksource/rockchip_timer.c`

## Corresponding registers and usage

1. The register is as follows:

![](../../../rk3399_img/include/RK3399/timer_register.jpg)

2. Use the `io` command to view the corresponding register:

```
root@host???:/ # io -4 0xff85001c  //View the status of the current control register
ff85001c:  00000007

root@host???:/ # io -4 0xff850000  //Check the current value of the register
ff850000:  0001639f
```

Control corresponding register:

```
root@host???:/ # io -4 -w 0xff85001c 0x06  //Turn off time counting
```
