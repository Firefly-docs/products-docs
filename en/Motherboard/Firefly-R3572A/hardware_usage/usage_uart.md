# UART Usage

## Introduction [step]

Firefly-R3572A provides two kinds of serial ports: the J3 Debug connector is the debug console using the m0 pin multiplexing of the UART0 controller, and UART2 is available on the J1 double-row pin header for application communication. Additional UART resources can be enabled in the SDK as required.

## Debug Console (UART0 m0) [step]

The J3 Debug connector is used as the system debug console by default at 1500000 baud. For wiring and usage, refer to [Debug Usage](usb_to_ttl.md).

<Callout title="Note" type="warn">

The debug console outputs system logs. Do not attach application devices to it, to avoid interference with the system log output.

</Callout>

## Expansion UART (UART2) [step]

UART2 is available on the J1 double-row pin header with the following pin definition:

| Pin NO. | Signal | Level |
| --- | --- | --- |
| 17 | UART2_TX (series resistance 22R) | 3.3 V |
| 19 | UART2_RX (series resistance 22R) | 3.3 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

## DTS Configuration [step]

Enable the UART2 controller in the board-level DTS (the pinctrl group depends on the SDK code):

```text
&uart2 {
    status = "okay";
    pinctrl-0 = <&uart2m2_xfer_pins>;
};
```

## Transceiver Test [step]

After the configuration, the system creates the serial device node `/dev/ttyS2` (the node number depends on the actual configuration). Short the UART2_TX and UART2_RX pins on the J1 header, then run the following commands on the debug console or through ADB:

```bash
busybox stty -echo -F /dev/ttyS2          # Disable echo
cat /dev/ttyS2 &                          # Read input in the background
echo "firefly uart test..." > /dev/ttyS2  # Send a string
```

Receiving the string `firefly uart test...` in the terminal confirms that the transceiver works.
