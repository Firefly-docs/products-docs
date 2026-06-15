# UART使用

<font color=#FF0000> 本章包含RS232节点和RS485节点的说明</font>

## 硬件

iCore-3576JQ的串口接口图如下：

![](../../../rk3576_img/iCore-3576JQ/usage_uart_interface.jpg)

## DTS配置

文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc.dtsi`
```
/* uart6 */
&uart6 {
        pinctrl-names = "default";
        pinctrl-0 = <&uart6m3_xfer>;
        status = "okay";
};
```
文件路径`kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-firefly-roc-rk3576-pc-ext.dtsi`
```
&spi3 {
        status = "okay";
        //max-freq = <48000000>;
        dev-port = <0>;
    pinctrl-0 = <&spi3m1_pins>;
        num-cs = <1>;

        spi_wk2xxx: spi_wk2xxx@00{
                status = "okay";
                compatible = "firefly,spi-wk2xxx";
                reg = <0x00>;
                spi-max-frequency = <10000000>;
                //power-gpio = <&gpio2 4 GPIO_ACTIVE_HIGH>;
                reset-gpio = <&gpio4 RK_PA0 GPIO_ACTIVE_HIGH>;
                irq-gpio = <&gpio3 RK_PA4 IRQ_TYPE_EDGE_FALLING>;
                cs-gpio = <&gpio3 RK_PD7 GPIO_ACTIVE_HIGH>;
        };
};
```

配置好串口后，硬件接口对应软件上的节点为：
```
RS485 节点:   /dev/ttysWK0(丝印：A1 B1)    /dev/ttysWK1(丝印：A2 B2)
RS232 节点:   /dev/ttysWK3(丝印：T1 R1)    /dev/ttysWK2(丝印：T2 R2)   /dev/ttyS6(丝印：T3 R3)
```

## 232 节点 收发验证

最简单的方式短接RS232 TX RX 引脚, 然后使用命令在调试串口或ADB执行命令

节点/dev/ttyS6测试示例如下（请根据实际短接脚位选择 /dev/ttysWK2， /dev/ttysWK3， /dev/ttyS6）
```
busybox  stty -echo -F /dev/ttyS6          # 关闭回显，
cat /dev/ttyS6 &                           # 后台获取/dev/ttyS7输入字符串
echo "firefly uart test..." > /dev/ttyS6   # 输入字符串
```
最终调试串口终端即可接收到字符串 "firefly uart test..."

## 485 节点 收发验证

最简单的方式两个RS485互相短接， A1接A2,B1接B2, 然后使用命令在调试串口或ADB执行命令

```
busybox  stty -echo -F /dev/ttysWK0          # 关闭回显
cat /dev/ttysWK1 &                           # 后台获取/dev/ttysWK1输入字符串
echo "firefly uart test..." > /dev/ttysWK0   # 输入字符串
```

最终调试串口终端即可接收到字符串 "firefly uart test..."
