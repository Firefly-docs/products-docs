# I2C 使用

## 简介 [step]

I2C 是一种两线制串行通信总线，用于连接低速外设。Firefly-R3572A 的 I2C 控制器分配如下：

| 控制器 | 用途 | 连接器 | 电平 |
| --- | --- | --- | --- |
| I2C2 | 对外扩展 | J1 排针（引脚 10 SDA / 13 SCL） | 1.8 V |
| I2C3 | 摄像头 0 | J20（MIPI-CSI0） | 1.8 V |
| I2C5 | 摄像头 1 | J21（MIPI-CSI1） | 1.8 V |
| I2C6 | 触摸屏 | J18（MIPI-DSI） | 3.3 V |

## 硬件连接 [step]

对外扩展使用 J1 排针上的 I2C2，引脚定义如下：

| 排针序号 | 信号 | 电平 |
| --- | --- | --- |
| 13 | I2C2_SCL_M2 | 1.8 V |
| 10 | I2C2_SDA_M2 | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="注意" type="warn">

J1 排针上的 I2C 电平为 1.8 V，外接 I2C 设备需使用 1.8 V 电平或加装电平转换电路。

</Callout>

## 查看设备节点 [step]

系统启动后确认 I2C 控制器对应的设备节点：

```bash
ls /dev/i2c-*
```

节点编号与控制器的对应关系可通过以下命令查看：

```bash
i2cdetect -l
```

## 扫描设备 [step]

扫描 I2C 总线上的设备地址（以下以 I2C2 为例，节点编号以系统实际为准）：

```bash
i2cdetect -y 2
```

已挂载的设备地址会显示 `UU`（被驱动占用）或具体地址值。

## 用户态读写 [step]

安装 `i2c-tools` 后，可使用 `i2cget`、`i2cset`、`i2cdump` 读写设备寄存器：

```bash
i2cget -f -y 2 0x51 0x00        # 读取设备 0x51 的 0x00 寄存器
i2cset -f -y 2 0x51 0x00 0x12   # 向设备 0x51 的 0x00 寄存器写入 0x12
```

## 内核驱动使用 [step]

内核驱动的 DTS 配置参考（节点名与外设以实际硬件为准）：

```text
&i2c2 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&i2c2m2_xfer>;

    demo_sensor: demo_sensor@51 {
        compatible = "vendor,demo-sensor";
        reg = <0x51>;
    };
};
```

DTS 文件路径与 pinctrl 宏名以 Firefly-R3572A SDK 实际代码为准。
