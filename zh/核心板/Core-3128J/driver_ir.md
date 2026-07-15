# IR 使用

## 红外遥控配置

Firefly-RK3128 开发板上使用红外收发传感器 IR (在 USB OTG 接口和音频接口之间)实现遥控功能。本文主要描述在开发板上如何配置红外遥控器。
其配置步骤可分为两个部分：
修改内核驱动：内核空间修改，Linux 和 Android 都要修改这部分的内容。
修改键值映射：用户空间修改（仅限 Android 系统）。
内核驱动

在 Linux 内核中，IR 驱动仅支持 NEC 编码格式。以下是在内核中配置红外遥控的方法。
所涉及到的文件：
dts 配置文件：kernel/arch/arm/boot/dts/rk3128-fireprime.dts
驱动源文件： kernel/drivers/input/remotectl/rk_pwm_remotectl.c
添加 IR 及对应键值表