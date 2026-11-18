# LED 使用

## 简介 [step]

Firefly-R3572A 板载 3 颗 LED，由 GPIO 直接驱动（DTS 节点 `firefly-leds`，`compatible = "gpio-leds"`），定义如下：

| LED | DTS label | GPIO | 默认触发方式 | 上电默认状态 |
| --- | --- | --- | --- | --- |
| 电源灯 | `:power` | GPIO0_PC0 | `ir-power-click` | 亮（on） |
| 用户灯 | `:user` | GPIO0_PC1 | `ir-user-click` | 灭（off） |
| DIY 灯 | `:diy` | GPIO3_PD6 | `none` | 灭（off） |

可通过 LED 子系统（sysfs）或直接操作 GPIO 控制。上表的节点名、GPIO 与 pinctrl 取自 Firefly-R3572A 的 Linux SDK 板级 DTS `rk3572-firefly-r3572a.dtsi`（由主 DTS 的 `N_LED` 开关控制是否启用），实际以你手中 SDK 代码为准。

## 以设备方式控制 LED [step]

Linux 的 LED 子系统在 `/sys/class/leds/` 下为每个 LED 生成目录，可通过 `brightness` 属性控制亮灭：

```bash
# 查看板上的 LED
ls /sys/class/leds/

# 点亮 / 熄灭 DIY 灯（1 亮、0 灭）
echo 1 > /sys/class/leds/:diy/brightness
echo 0 > /sys/class/leds/:diy/brightness
```

查看当前可使用（或正在使用）的触发方式：

```bash
cat /sys/class/leds/:diy/trigger
```

## 使用 trigger 控制 LED [step]

LED 的 trigger 可以实现自动闪烁、按事件点亮等效果。例如让 DIY 灯按定时器周期闪烁：

```bash
# 确认内核已选中 LED Timer Trigger（Device Drivers > LED Support > LED Trigger support > LED Timer Trigger）
echo timer > /sys/class/leds/:diy/trigger
```

设置为心跳灯等其它触发方式同理，例如：

```bash
echo heartbeat > /sys/class/leds/:diy/trigger
```

取消触发、恢复手动控制：

```bash
echo none > /sys/class/leds/:diy/trigger
```

## DTS 配置参考 [step]

Firefly-R3572A 的 LED 节点定义（位于 `kernel-6.12/arch/arm64/boot/dts/rockchip/rk3572-firefly-r3572a.dtsi`）：

```text
firefly_leds: firefly-leds {
    compatible = "gpio-leds";
    status = "okay";
    power_led: power {
        label = ":power";
        linux,default-trigger = "ir-power-click";
        gpios = <&gpio0 RK_PC0 GPIO_ACTIVE_HIGH>;
        default-state = "on";
    };
    user_led: user {
        label = ":user";
        linux,default-trigger = "ir-user-click";
        gpios = <&gpio0 RK_PC1 GPIO_ACTIVE_HIGH>;
        default-state = "off";
    };
    diy_led: diy {
        label = ":diy";
        linux,default-trigger = "none";
        gpios = <&gpio3 RK_PD6 GPIO_ACTIVE_HIGH>;
        default-state = "off";
    };
};
```

<Callout title="注意" type="info">

`compatible = "gpio-leds"` 需与驱动 `kernel-6.12/drivers/leds/leds-gpio.c` 中的 `.compatible` 保持一致。修改默认触发方式或新增 LED 时，请同步修改板级 DTS 并重新编译烧录。

</Callout>
