# PWM 使用

## 简介 [step]

PWM（脉冲宽度调制）常用于背光调节、风扇调速、蜂鸣器与舵机控制等场景。Firefly-R3572A 的 PWM 驱动为 SDK 内核的 `pwm-rockchip` 驱动，PWM 信号通过 J18（MIPI-DSI 连接器）的 `LCD_BL_PWM` 引脚与 J1 双排排针引出。

## 硬件连接 [step]

| 连接器 | 信号 | 电平 |
| --- | --- | --- |
| J18 引脚 12 | LCD_BL_PWM | 3.3 V |
| J1 双排排针 | PWM | 以实际配置为准 |

## DTS 配置 [step]

Firefly-R3572A SDK 中已使用一个 6 通道 PWM 控制器作为 MIPI 屏背光，位于 `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a-mipi101-BSD1218-A101KL68.dtsi`：

```text
&backlight {
    compatible = "pwm-backlight";
    pwms = <&pwm1_6ch_5 0 50000 1>;
};

&pwm1_6ch_5 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&pwm1m0_ch5_pins>;
};
```

若要新增自己的 PWM 设备，可在板级 DTS 中按同样的写法申请通道（控制器、通道号与 pinctrl 宏以 SDK 实际代码为准）：

```text
pwm_demo: pwm_demo {
    compatible = "firefly,pwm-demo";
    status = "okay";
    pwms = <&pwmX 0 25000 0>;
    duty-cycle-ratio = <50>;
    pinctrl-names = "default";
    pinctrl-0 = <&pwmXmX_pins>;
};
```

参数说明：

* `&pwmX`：PWM 控制器节点（如上述 `&pwm1_6ch_5`）
* `0`：通道编号
* `25000`：周期，单位 ns（对应 40 kHz）
* `duty-cycle-ratio`：占空比，百分比

## sysfs 使用 [step]

系统启动后，通过 sysfs 接口控制 PWM：

```bash
echo 0 > /sys/class/pwm/pwmchipX/export      # 申请通道 0
echo 25000 > /sys/class/pwm/pwmchipX/pwm0/period      # 设置周期 25 us
echo 12500 > /sys/class/pwm/pwmchipX/pwm0/duty_cycle  # 设置占空时间 12.5 us
echo normal > /sys/class/pwm/pwmchipX/pwm0/polarity   # 设置极性
echo 1 > /sys/class/pwm/pwmchipX/pwm0/enable          # 使能输出
```

`pwmchipX` 的编号以系统实际枚举为准，可通过 `ls /sys/class/pwm/` 确认。

停止输出：

```bash
echo 0 > /sys/class/pwm/pwmchipX/pwm0/enable
```

## 调试方法 [step]

用示波器或逻辑分析仪测量对应引脚，确认输出波形频率与占空比与配置一致。如无输出，依次检查：

1. DTS 中 PWM 节点 `status` 是否为 `okay`。
2. pinctrl 是否将引脚复用为 PWM 功能。
3. `duty_cycle` 是否小于 `period`。
## 接口说明 [step]

用户可在其它驱动文件中使用以上步骤生成的 PWM 节点。具体方法如下：

(1)、在要使用 PWM 控制的设备驱动文件中包含以下头文件：

```c
#include <linux/pwm.h>
```

该头文件主要包含 PWM 的函数接口。

(2)、申请 PWM

使用

```c
struct pwm_device *devm_pwm_get(struct device *dev, const char *con_id);
```

函数申请 PWM。例如：

```c
struct pwm_device *pwm0 = devm_pwm_get(&pdev->dev, NULL);
```

(3)、配置 PWM

使用

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

配置 PWM 的占空比，例如：

```c
pwm_config(pwm0, 500000, 1000000);
```

(4)、使能 PWM

```c
int pwm_enable(struct pwm_device *pwm);
```

用于使能 PWM，例如：

```c
pwm_enable(pwm0);
```

(5)、控制 PWM 输出主要使用以下接口函数：

* 功能：用于申请 PWM

```c
struct pwm_device *devm_pwm_get(struct device *dev, const char *con_id);
```

* 功能：用于释放所申请的 PWM

```c
void pwm_free(struct pwm_device *pwm);
```

* 功能：用于配置 PWM 的占空比

```c
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```

* 功能：使能 PWM

```c
int pwm_enable(struct pwm_device *pwm);
```

* 功能：禁止 PWM

```c
void pwm_disable(struct pwm_device *pwm);
```

参考例子：`kernel-6.12/drivers/pwm/pwm-firefly-demo.c`

## FAQs [step]

### PWM 无法注册成功：

* dts 配置文件是否打开对应的 PWM。
* PWM 所在的 IO 口是否被其他资源占用，可以根据报错的返回值去查看原因。
