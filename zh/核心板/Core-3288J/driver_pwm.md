# PWM 使用

## 前言

AIO-3288J 开发板上有 4 路 PWM 输出，分别为 PWM0 ~ PWM3，本章主要描述如何配置 PWM。

PWM 驱动文件：

```c
kernel/drivers/pwm/pwm-rockchip.c
```

## 数据结构

### pwm_device结构体

```c
struct pwm_device {
    const char      *label;
    unsigned long       flags;
    unsigned int        hwpwm;
    unsigned int        pwm;//pwm通道
    struct pwm_chip     *chip;
    void            *chip_data;
    unsigned int        period; /* in nanoseconds */
};
```

### pwm_chip结构体

该结构体是抽象的 PWM 控制器。

```c
struct pwm_chip {
    struct device       *dev; //提供 PWM 的设备
    struct list_head    list; //内部使用的节点列表
    const struct pwm_ops    *ops; //该 PWM 控制器的回调函数
    int  base; //该设备所控制的第一个 PWM 的号码
    unsigned int        npwm; //该设备所控制的 PWM 数
    struct pwm_device   *pwms;
    struct pwm_device * (*of_xlate)(struct pwm_chip *pc,const struct of_phandle_args *args);
    unsigned int of_pwm_n_cells;
    bool can_sleep;
};
```

## 配置步骤

配置 PWM 主要有以下三大步骤：配置 PWM DTS 节点、配置 PWM 内核驱动、控制 PWM 设备。  

### 配置 PWM DTS节点

在 `kernel/arch/arm/boot/dts/rk3288.dtsi` 定义了以下 PWM 节点，如下所示：

```dts
pwm1: pwm@ff680010 {
      compatible = "rockchip,rk-pwm";
      reg = ;
      #pwm-cells = ;
      pinctrl-names = "default";
      pinctrl-0 = ;
      clocks = ;
      clock-names = "pclk_pwm";
      status = "okay";
};
```

注：ff680010为 PWM1 寄存器的地址。

要使用 pwm1, 只需在 `kernel/arch/arm/boot/dts/firefly-rk3288.dts` 加入：

```dts
&pwm0 {
    status = "okay";
};
```

## 配置 PWM 内核驱动

PWM 驱动位于文件 `kernel/drivers/pwm/pwm-rockchip.c`。

修改该文件的如下代码：

```c
static const struct of_device_id rk_pwm_of_match[] = {
      { .compatible = "rockchip,pwm",       .data = &rk_pwm_data_v1,},
      { .compatible =  "rockchip,rk-pwm",   .data = &rk_pwm_data_v2,},
      { .compatible =  "rockchip,vop-pwm",  .data = &rk_pwm_data_v3,},
      { }
};
```

把之前步骤中的 compatible = "rockchip,rk-pwm" 添加到如上代码中。

## 控制 PWM 设备

用户可在其它驱动文件中使用以上步骤生成的 PWM 节点。具体方法如下：

1、在要使用 PWM 控制的设备驱动文件中包含以下头文件：

```c
#include <linux/pwm.h>
```

该头文件主要包含 PWM 的函数接口。

2、申请 PWM 使用

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

函数申请 PWM。例如：

```c
struct pwm_device * pwm0 = NULL;pwm0 = pwm_request(0, “backlight-pwm”);
```

参数 pwm_id 表示要申请 PWM 的通道，label 为该 PWM 所取的标签。

3、配置 PWM 使用

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

配置 PWM 的占空比，例如：

```c
pwm_config(pwm0, 500000, 1000000)；
```

参数 pwm 为前一步骤申请的 pwm_device。duty_ns 为占空比激活的时长，单位为 ns。period_ns 为 PWM 周期，单位为 ns。

4、使能PWM函数

```c
int pwm_enable(struct pwm_device *pwm);
```

用于使能 PWM，例如：

```c
pwm_enable(pwm0);
```

参数 pwm 为要使能的 pwm_device。

控制 PWM 输出主要使用以下接口函数：

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

* 功能：用于申请 pwm
* 参数：
    + pwm_id：要申请的 pwm 通道。
    + label: 为该申请的 pwm 所取的标签。

```c
void pwm_free(struct pwm_device *pwm);
```

* 功能：用于释放所申请的 pwm
* 参数：
    + pwm：所要释放的 pwm 结构体

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

* 功能：用于配置 pwm 的占空比
* 参数：
    + pwm: 所要配置的 pwm
    + duty_ns：pwm 的占空比激活的时长，单位 ns
    + period_ns：pwm 占空比周期，单位 ns

```c
int pwm_enable(struct pwm_device *pwm);
```

* 功能：使能 pwm
* 参数：
    + pwm：要使能的 pwm

```c
void pwm_disable(struct pwm_device *pwm);
```

* 功能：禁止 pwm
* 参数：
    + pwm：要禁止的 pwm