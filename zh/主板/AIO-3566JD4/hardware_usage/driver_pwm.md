# PWM 使用

## 前言 [step]
本章主要描述如何配置 PWM。

AIO-3566JD4 的 PWM 驱动为：`kernel/drivers/pwm/pwm-rockchip.c`

## DTS配置 [step]

配置 PWM 主要有以下三大步骤：配置 PWM DTS 节点、配置 PWM 内核驱动、控制 PWM 设备。

### 配置 PWM DTS节点

在 DTS 源文件 `kernel/arch/arm64/boot/dts/rockchip/rk356x-firefly-demo.dtsi` 添加 PWM DTS 配置，如下所示：

```dts
pwm_demo: pwm_demo {
        status = "disabled";
        compatible = "firefly,rk356x-pwm";
        pwms = <&pwm1 0 10000 1>;   //pwm1:PWM number   0 10000:PWM period in nanoseconds  1:polarity
        duty_ns = <5000>;   //pwm duty cycle activation time, unit ns
    };
```

## 接口说明 [step]

用户可在其它驱动文件中使用以上步骤生成的 PWM 节点。具体方法如下：

（1）在要使用 PWM 控制的设备驱动文件中包含以下头文件：

```c
#include <linux/pwm.h>
```

该头文件主要包含 PWM 的函数接口。

（2）申请 PWM

使用

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

函数申请 PWM。例如：

```c
struct pwm_device * pwm1 = NULL;pwm0 = pwm_request(1, “firefly-pwm”);
```

（3）配置 PWM

使用

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

配置 PWM 的占空比，例如：

```c
pwm_config(pwm0, 500000, 1000000);
```

（4）使能 PWM 函数

```c
int pwm_enable(struct pwm_device *pwm);
```

用于使能 PWM，例如：

```c
pwm_enable(pwm0);
```

（5）控制 PWM 输出主要使用以下接口函数：

- 功能：用于申请 PWM

```c
struct pwm_device *pwm_request(int pwm_id, const char *label);
```

- 功能：用于释放所申请的 PWM

```c
void pwm_free(struct pwm_device *pwm);
```

- 功能：用于配置 PWM 的占空比

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

- 功能：使能 PWM

```c
int pwm_enable(struct pwm_device *pwm);
```

- 功能：禁止 PWM

```c
void pwm_disable(struct pwm_device *pwm);
```

参考例子：`kernel/drivers/pwm/pwm-firefly.c`

## 调试方法 [step]

通过内核丰富的 debug 接口查看 PWM 注册状态，`adb shell` 或者串口进入 Android 终端执行：

```bash
cat  /sys/kernel/debug/pwm
```

查看注册是否成功，成功则返回接口名和寄存器地址。

## FAQs [step]

### PWM 无法注册成功：

- dts 配置文件是否打开对应的 PWM。
- PWM 所在的 IO 口是否被其他资源占用，可以根据报错的返回值去查看原因。
