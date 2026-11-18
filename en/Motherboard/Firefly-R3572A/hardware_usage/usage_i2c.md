# I2C Usage

## Introduction [step]

I2C is a two-wire serial bus for connecting low-speed peripherals. The I2C controllers of Firefly-R3572A are allocated as follows:

| Controller | Usage | Connector | Level |
| --- | --- | --- | --- |
| I2C2 | External expansion | J1 header (pin 10 SDA / 13 SCL) | 1.8 V |
| I2C3 | Camera 0 | J20 (MIPI-CSI0) | 1.8 V |
| I2C5 | Camera 1 | J21 (MIPI-CSI1) | 1.8 V |
| I2C6 | Touch panel | J18 (MIPI-DSI) | 3.3 V |

## Hardware Connection [step]

For external expansion, use I2C2 on the J1 header:

| Pin NO. | Signal | Level |
| --- | --- | --- |
| 13 | I2C2_SCL_M2 | 1.8 V |
| 10 | I2C2_SDA_M2 | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="Note" type="warn">

The I2C level on the J1 header is 1.8 V. Use 1.8 V-level devices or add a level shifter for external I2C devices.

</Callout>

## Checking the Device Node [step]

After the system boots, check the device nodes of the I2C controllers:

```bash
ls /dev/i2c-*
```

The mapping between node numbers and controllers can be checked with:

```bash
i2cdetect -l
```

## Scanning Devices [step]

Scan the device addresses on the I2C bus (I2C2 as an example; the node number depends on the system):

```bash
i2cdetect -y 2
```

Connected devices appear as `UU` (occupied by a driver) or as an address value.

## User-Space Access [step]

After installing `i2c-tools`, use `i2cget`, `i2cset` and `i2cdump` to access device registers:

```bash
i2cget -f -y 2 0x51 0x00        # Read register 0x00 of device 0x51
i2cset -f -y 2 0x51 0x00 0x12   # Write 0x12 to register 0x00 of device 0x51
```

## Using I2C in a Kernel Driver [step]

Example DTS configuration for a kernel driver (node names and peripherals depend on the actual hardware):

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

Refer to the actual Firefly-R3572A SDK code for the DTS file path and the pinctrl macro names.
