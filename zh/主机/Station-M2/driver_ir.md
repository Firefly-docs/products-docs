# IR 使用

## 红外遥控配置



其配置步骤可分为两个部分：

*    修改内核驱动：内核空间修改，Linux 和 Android 都要修改这部分的内容。
*    修改键值映射：用户空间修改（仅限 Android 系统）。

## 内核驱动

在 Linux 内核中，IR 驱动仅支持 NEC 编码格式。以下是在内核中配置红外遥控的方法。

Android系统kernel所涉及到的文件：

```
kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-port.dtsi
kernel/drivers/input/remotectl/rockchip_pwm_remotectl.c
```

#### 定义相关数据结构

键值表结构体数组和配置参考如下：

```
&pwm7 {
        status = "okay";

        compatible = "rockchip,remotectl-pwm";
        interrupts = <GIC_SPI 83 IRQ_TYPE_LEVEL_HIGH>;
        remote_pwm_id = <3>;
        handle_cpu_id = <1>;
        remote_support_psci = <0>;
        pinctrl-names = "default";
        pinctrl-0 = <&pwm7_pins>;

        ir_key_firefly{
                rockchip,usercode = <0xff00>;
                rockchip,key_table =
                        <0xeb   KEY_POWER>,
                        <0xec   KEY_MENU>,
                        <0xfe   KEY_BACK>,
                        <0xb7   KEY_HOME>,
                        <0xa3   KEY_WWW>,
                        <0xf4   KEY_VOLUMEUP>,
                        <0xa7   KEY_VOLUMEDOWN>,
                        <0xf8   KEY_REPLY>,
                        <0xfc   KEY_UP>,
                        <0xfd   KEY_DOWN>,
                        <0xf1   KEY_LEFT>,
                        <0xe5   KEY_RIGHT>;
        };
};
```

注：rockchip,key_table第一列为键值，第二列为要响应的按键码。

#### 如何获取用户码和IR 键值

在rockchip_pwm_remotectl.c驱动中的 remotectl_do_something 函数获取用户码和键值：

```
    case RMC_USERCODE: {
                if ((RK_PWM_TIME_BIT1_MIN < ddata->period) &&
                    (ddata->period < RK_PWM_TIME_BIT1_MAX))
                        ddata->scandata |= (0x01 << ddata->count);
                ddata->count++;
                if (ddata->count == 0x10) {
                        DBG_CODE("USERCODE=0x%x\n", ddata->scandata);
                        if (remotectl_keybd_num_lookup(ddata)) {
                                ddata->state = RMC_GETDATA;
                                ddata->scandata = 0;
                                ddata->count = 0;
                        } else {
                                if (rk_remote_print_code){
                                        ddata->state = RMC_GETDATA;
                                        ddata->scandata = 0;
                                        ddata->count = 0;
                                } else
                                        ddata->state = RMC_PRELOAD;
                        }
                }
        }

```

注：用户可以使用 `DBG_CODE()` 函数打印用户码。

开发板使用下面命令可以使能 DBG_CODE 打印：

```
echo 1 > /sys/module/rockchip_pwm_remotectl/parameters/code_print
```

#### 将 IR 驱动编译进内核

由于SDK中已经存在IR驱动rockchip_pwm_remotectl.c，我们需要做的就是将 IR 驱动编译进内核，在Android系统中可以添加以下两条语句到`kernel/arch/arm64/configs/firefly_defconfig `文件

```
CONFIG_ROCKCHIP_REMOTECTL=y
CONFIG_ROCKCHIP_REMOTECTL_PWM=y
```

保存后，按照固件编译的方法编译kernel即可

* [编译 Android 固件](compile_android11.0_firmware.html)

#### Android 键值映射

文件 `/system/usr/keylayout/ff420030_pwm.kl` 用于将 Linux 层获取的键值映射到 Android 上对应的键值。用户可以添加或者修改该文件的内容以实现不同的键值映射。

该文件内容如下所示：
```
key 28    ENTER
key 116   POWER             WAKE
key 158   BACK
key 139   MENU
key 217   SEARCH
key 232   DPAD_CENTER
key 108   DPAD_DOWN
key 103   DPAD_UP
key 102   HOME
key 105   DPAD_LEFT
key 106   DPAD_RIGHT
key 115   VOLUME_UP
key 114   VOLUME_DOWN
key 143   NOTIFICATION      WAKE
key 113   VOLUME_MUTE
key 388   TV_KEYMOUSE_MODE_SWITCH
key 400   TV_MEDIA_MULT_BACKWARD
key 401   TV_MEDIA_MULT_FORWARD
key 402   TV_MEDIA_PLAY_PAUSE
key 64    TV_MEDIA_PLAY
key 65    TV_MEDIA_PAUSE
key 66    TV_MEDIA_STOP
```

注：通过 ADB 修改该文件重启后即可生效。

## IR 使用

如下图是通过按红外遥控器按钮，所产生的波形，主要由 head, Control, information, signed free 这四部分组成，具体可以参考 RC6 Protocol。

![](../../../rk356x_img/ir.jpg)
