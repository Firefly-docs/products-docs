# PWM 使用

## 前言
AIO-3399C 开发板上引出有 3 路 PWM 输出，分别为：

* PWM0 屏背光
* PWM2 VDDLOG 供电
* PWM3 红外 IR

本章主要描述如何配置 PWM。

AIO-3399C 的 PWM 驱动为：`kernel/drivers/pwm/pwm-rockchip.c`

## DTS配置

配置 PWM 主要有以下三大步骤：配置 PWM DTS 节点、配置 PWM 内核驱动、控制 PWM 设备。

### 配置 PWM DTS节点

在 DTS 源文件 `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi` 添加 PWM DTS 配置，如下所示：

```
pwm_demo: pwm_demo {
    status = "okay";
    compatible = "firefly,rk3399-pwm";
    pwm_id = <1>;
    min_period = <0>;
    max_period = <10000>;
    duty_ns = <5000>;
};
```

* pwm_id：需要申请的 PWM 通道数。
* min_period：周期时长最小值。
* max_period：周期时长最大值。
* duty_ns：PWM 的占空比激活的时长，单位 ns。

## 接口说明

用户可在其它驱动文件中使用以上步骤生成的 PWM 节点。具体方法如下：

(1)、在要使用 PWM 控制的设备驱动文件中包含以下头文件：

```
#include <linux/pwm.h>
```

该头文件主要包含 PWM 的函数接口。

(2)、申请 PWM

使用

```
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

函数申请 PWM。 例如：

```
struct pwm_device * pwm1 = NULL;pwm0 = pwm_request(1, “firefly-pwm”);
```

(3)、配置 PWM

使用

```
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

配置 PWM 的占空比，例如：

```
pwm_config(pwm0, 500000, 1000000);
```

(4)、使能PWM函数

```
int pwm_enable(struct pwm_device *pwm);
```

用于使能 PWM，例如：

```
pwm_enable(pwm0);
```

(5)控制 PWM 输出主要使用以下接口函数：

* 功能：用于申请 PWM

```
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

* 功能：用于释放所申请的 PWM

```
void pwm_free(struct pwm_device *pwm);
```

* 功能：用于配置 PWM 的占空比

```
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

* 功能：使能 PWM

```
int pwm_enable(struct pwm_device *pwm);
```

* 功能：禁止 PWM

```
void pwm_disable(struct pwm_device *pwm);
```

参考例子： `kernel/drivers/pwm/pwm-firefly.c`

## 调试方法

通过内核丰富的 debug 接口查看 PWM 注册状态，`adb shell` 或者串口进入 Android 终端执行：

```
cat  /sys/kernel/debug/pwm
```

查看注册是否成功，成功则返回接口名和寄存器地址。

## FAQs

### PWM 无法注册成功：

* dts 配置文件是否打开对应的 PWM。
* PWM 所在的 IO 口是否被其他资源占用，可以根据报错的返回值去查看原因。
