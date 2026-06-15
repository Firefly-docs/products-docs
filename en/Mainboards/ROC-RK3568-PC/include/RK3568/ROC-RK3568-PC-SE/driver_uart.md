
# UART

## Introduction

ROC-RK3568-PC supports UART、RS232、RS485 interfaces. They are UART7, two RS232 and one RS485, which are located on the dual expansion interface and RJ45 interface respectively. The serial port interface diagram is as follows:

![](../../img/ROC-RK3568-PC/uart_interface.jpg)

**Note**: In this chapter, two RS232 will be defined as `RS232_1`、`RS232_2`, which is used to explains how to use UART.


## Pin definition

### UART7

TX and RX of UART7 are multiplexed. Actually，`UART7 TX` and `UART7 RX` correspond to the silk screen printing `PWM14` and `SPDIF` on the development board respectively. UART9 is turned on by default in the public firmware. If you want to using other functions, UART9 needs to be turned off first. The details is follow:

| func0 | func1 | func2 | func3 | func4 | func5 |
| --- | --- | --- | --- | --- | --- |
| GPIO3_C4 | PWM14_M0 | VOP_PWM_M1 | GMAC1_MDC_M0 | UART7_TX_M1 | PDM_CLK1_M2 |
| GPIO3_C5 | SPDIF_TX_M1 | PWM15_IR_M0 | GMAC1_MDIO_M0 | UART7_RX_M1 | I2S1_LRCK_RX_M2 |

### RS232 and RS485

RS232_1,RS232_2 and RS485 are respectively converted from UART2, UART3 and UART4 of the main control.The RS232_1 cannot be used directly because UART2 is used as debug serial port by default. If want to use RS232_1, you need to [configure UART2 as a normal serial port](#how-to-configure-uart2-as-a-normal-serial-port).For relevant hardware connection and definition, you can refer [ROC-RK3568-PC schematic diagram](https://en.t-firefly.com/doc/download/157.html#other_558). The following are some pins definition of RJ45 interface:

| RJ45 PINS |    Definition    | RJ45 PINS |    Definition    |
| :------: | :--------: | :------: | :--------: |
|    1     | RS232_2 TX |    5     |    GND     |
|    2     | RS232_2 RX |    6     | RS232_1 RX |
|    3     | RS232_1 TX |    7     |  RS485_A   |
|    4     |    GND     |    8     |  RS485_B   |

## DTS configuration

Because UART2 is the debug serial port by default, the RS232_1 converted from UART2 cannot be used directly. We only need to configure in `kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc.dtsi`:

* RS232_2
    * the corresponding node `/dev/ttyS3`
    ```
    &uart3 {
	    status = "okay";
	    pinctrl-0 = <&uart3m1_xfer>;
    };
   ```

* RS485
    * the corresponding node `/dev/ttyS4`
    ```
    &uart4 {
	    status = "okay";
	    pinctrl-0 = <&uart4m1_xfer>;
    };
    ```

* UART7
    * the corresponding node `/dev/ttyS7`
    ```
    &uart7 {
	    status = "okay";
	    pinctrl-0 = <&uart7m1_xfer>;
    };
    ```

## Debug method

Users can send and receive data to the serial port of the development board by using the USB to serial port adapter of different host according to different interfaces. For example, **the debugging steps of RS485 are as follows**:

(1) Connected hardware

Find the RJ45 interface with RS485 on the development board, and connect 5(GND), 7(A) and 8(B) pins of the RJ45 with GND, A and B pins of the host serial port adapter (USB to 485 to serial port module) respectively.


(2) Open the serial terminal of the host

Input the install Kermit command `sudo apt install Kermit` on the terminal. After the installation, open Kermit, set the baud rate and connect:

```
$ sudo kermit
C-Kermit> set line /dev/ttyUSB0
C-Kermit> set speed 9600
C-Kermit> set flow-control none
C-Kermit> connect
```

* `/dev/ttyUSB0` is the device file for USB to serial port adapter.

(3) Send data to host serial port adapter

The device file for RS485 is `/dev/ttyS4`. Run the following commands on the device:

```
echo firefly RS485 test... > /dev/ttyS4
```

The serial terminal in the host can receive a string: "firefly RS485 test... "

(4) Receive data from host serial port adapter

First run the following commands on the device:

```
cat /dev/ttyS4
```

Then enter a string at the host's serial terminal: `Firefly RS485 test...`, the device side can see the same string.

(5) Exit kermit connection

The host hit keyboard `ctrl+\` and then hit key c, input "exit" and hit enter

```
C-Kermit>exit
OK to exit? ok
```

## How to configure UART2 as a normal serial port

In the public firmware, UART2 defaults to debug serial port. Next, take Android platform as an example to modify UART2 to normal serial port.

### Kernel modification

* Remove `CONFIG_SERIAL_8250_CONSOLE` configuration in `kernel/arch/arm64/configs/firefly_defconfig`
```
diff --git a/kernel/arch/arm64/configs/firefly_defconfig b/kernel/arch/arm64/configs/firefly_defconfig
index 57ed787..8d6bc18 100644
--- a/kernel/arch/arm64/configs/firefly_defconfig
+++ b/kernel/arch/arm64/configs/firefly_defconfig
@@ -500,7 +500,7 @@ CONFIG_INPUT_RK805_PWRKEY=y
 # CONFIG_LEGACY_PTYS is not set
 CONFIG_SERIAL_8250=y
 # CONFIG_SERIAL_8250_DEPRECATED_OPTIONS is not set
-CONFIG_SERIAL_8250_CONSOLE=y
+#CONFIG_SERIAL_8250_CONSOLE=y
 # CONFIG_SERIAL_8250_PCI is not set
 CONFIG_SERIAL_8250_NR_UARTS=10
 CONFIG_SERIAL_8250_RUNTIME_UARTS=10
```


* Close the `fiq-debugger` node in `kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi`
```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi b/kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi
index 55a1716..0e297e6 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-android.dtsi
@@ -26,7 +26,7 @@
                interrupts = <GIC_SPI 252 IRQ_TYPE_LEVEL_LOW>;
                pinctrl-names = "default";
                pinctrl-0 = <&uart2m0_xfer>;
-               status = "okay";
+               status = "disabled";
        };

        debug: debug@fd904000 {

```

* Open UART2
```
diff --git a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc.dtsi b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc.dtsi
index f4af38a..fb9a3ff 100644
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3568-firefly-roc-pc.dtsi
@@ -137,6 +137,10 @@
        status = "okay";
 };

+&uart2 {
+       status = "okay";
+};
+
```

Recompile the kernel and upgrad it to the development board. After it takes effect, will generate `/dev/ttyS2` node.


