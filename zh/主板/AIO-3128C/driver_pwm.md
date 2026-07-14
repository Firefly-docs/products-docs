# PWM 使用

## 前言

AIO-3128C 开发板上有 4 路 PWM 输出，分别为 PWM0 ~ PWM3，其中：

* PWM0/GPIO0_D2: 在扩展口中引出
* PWM1/GPIO0_D3: 内部用作 AUX_DET 信号
* PWM2/GPIO0_D4: 内部用作 RTC_INT 信号

本章主要描述如何配置 PWM。
AIO-3128C 的 PWM 驱动为：

```
kernel/drivers/pwm/pwm-rockchip.c
```

## 配置步骤

配置 PWM 主要有以下三大步骤：配置 PWM DTS 节点、配置 PWM 内核驱动、控制 PWM 设备。  

配置 PWM DTS节点

在 kernel/arch/arm/boot/dts/rk312x.dtsi 定义了以下 PWM 节点，如下所示：

```
pwm0: pwm@20050000 {compatible = "rockchip,rk-pwm";reg = <0x20050000 0x10>;#pwm-cells = <2>;pinctrl-names = "default";pinctrl-0 = <&pwm0_pin>;clocks = <&clk_gates7 10>;clock-names = "pclk_pwm";status = "disabled";}; 
pwm1: pwm@20050010 {compatible = "rockchip,rk-pwm";reg = <0x20050010 0x10>;#pwm-cells = <2>;pinctrl-names = "default";pinctrl-0 = <&pwm1_pin>;clocks = <&clk_gates7 10>;clock-names = "pclk_pwm";status = "disabled";}; 
pwm2: pwm@20050020 {compatible = "rockchip,rk-pwm";reg = <0x20050020 0x10>;#pwm-cells = <2>;pinctrl-names = "default";pinctrl-0 = <&pwm2_pin>;clocks = <&clk_gates7 10>;clock-names = "pclk_pwm";status = "disabled";}; 
pwm3: pwm@20050030 {compatible = "rockchip,rk-pwm";reg = <0x20050030 0x10>;#pwm-cells = <2>;pinctrl-names = "default";pinctrl-0 = <&pwm3_pin>;clocks = <&clk_gates7 10>;clock-names = "pclk_pwm";status = "disabled";}; 
remotectl: pwm@20050030 {compatible = "rockchip,remotectl-pwm";reg = <0x20050030 0x10>;#pwm-cells = <2>;pinctrl-names = "default";pinctrl-0 = <&pwm3_pin>;clocks = <&clk_gates7 10>;clock-names = "pclk_pwm";remote_pwm_id = <3>;interrupts = <GIC_SPI 30 IRQ_TYPE_LEVEL_HIGH>;status = "okay";};
```

要使用 pwm0, 只需在 kernel/arch/arm/boot/dts/aio-3128c.dts 加入：

```
&pwm0 {
  status = "okay";
};
```

## 配置 PWM 内核驱动

PWM 驱动位于文件 kernel/drivers/pwm/pwm-rockchip.c。

## 控制 PWM 设备

用户可在其它驱动文件中使用以上步骤生成的 PWM 节点。具体方法如下：

### (1)、在要使用 PWM 控制的设备驱动文件中包含以下头文件：

```
#include <linux/pwm.h>
```

该头文件主要包含 PWM 的函数接口。

### (2)、申请 PWM

使用

```
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

函数申请 PWM。例如：

```
struct pwm_device * pwm0 = NULL;pwm0 = pwm_request(0, “backlight-pwm”);
```

参数 pwm_id 表示要申请 PWM 的通道，label 为该 PWM 所取的标签。

### (3)、配置 PWM

使用

```
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

配置 PWM 的占空比，例如：

```
pwm_config(pwm0, 500000, 1000000)；
```

参数 pwm 为前一步骤申请的 pwm_device。duty_ns 为占空比激活的时长，单位为 ns。period_ns 为 PWM 周期，单位为 ns。

### (4)、使能PWM

函数
```
int pwm_enable(struct pwm_device *pwm);
```

用于使能 PWM，例如：

```
pwm_enable(pwm0);
```

参数 pwm 为要使能的 pwm_device。

控制 PWM 输出主要使用以下接口函数：

```
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

* 功能：用于申请 pwm
* 参数：
  * pwm_id：要申请的 pwm 通道。
  * label: 为该申请的 pwm 所取的标签。
```
void pwm_free(struct pwm_device *pwm);
```

* 功能：用于释放所申请的 pwm
* 参数：
  * pwm：所要释放的 pwm 结构体
```
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

* 功能：用于配置 pwm 的占空比
* 参数：
  * pwm: 所要配置的 pwm
  * duty_ns：pwm 的占空比激活的时长，单位 ns
  * period_ns：pwm 占空比周期，单位 ns
```
int pwm_enable(struct pwm_device *pwm);
```

* 功能：使能 pwm
* 参数：
  * pwm：要使能的 pwm
```
void pwm_disable(struct pwm_device *pwm);        
```

* 功能：禁止 pwm
* 参数：
  * pwm：要禁止的 pwm