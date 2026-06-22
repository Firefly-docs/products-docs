# I2C

## Introduction

AIO-1126BJD4 have 6 on-chip I2C controllers on the development board.

`kernel-6.1/arch/arm64/boot/dts/rockchip/rv1126bjd4.dtsi`

        i2c0: i2c@21100000 {
            compatible = "rockchip,rv1126b-i2c";
            reg = <0x21100000 0x1000>;
            interrupts = <GIC_SPI 48 IRQ_TYPE_LEVEL_HIGH>;
            #address-cells = <1>;
            #size-cells = <0>;
            clocks = <&cru CLK_I2C0>, <&cru PCLK_I2C0>;
            clock-names = "i2c", "pclk";
            pinctrl-names = "default";
            pinctrl-0 = <&i2c0m0_pins>;
            status = "disabled";
        };

        i2c1: i2c@21110000 {
            compatible = "rockchip,rv1126b-i2c";
            reg = <0x21110000 0x1000>;
            interrupts = <GIC_SPI 49 IRQ_TYPE_LEVEL_HIGH>;
            #address-cells = <1>;
            #size-cells = <0>;
            clocks = <&cru CLK_I2C1>, <&cru PCLK_I2C1>;
            clock-names = "i2c", "pclk";
            dmas = <&dmac 31>, <&dmac 30>;
            dma-names = "tx", "rx";
            pinctrl-names = "default";
            pinctrl-0 = <&i2c1m0_pins>;
            status = "disabled";
        };

        i2c3: i2c@21120000 {
            compatible = "rockchip,rv1126b-i2c";
            reg = <0x21120000 0x1000>;
            interrupts = <GIC_SPI 51 IRQ_TYPE_LEVEL_HIGH>;
            #address-cells = <1>;
            #size-cells = <0>;
            clocks = <&cru CLK_I2C3>, <&cru PCLK_I2C3>;
            clock-names = "i2c", "pclk";
            dmas = <&dmac 35>, <&dmac 34>;
            dma-names = "tx", "rx";
            pinctrl-names = "default";
            pinctrl-0 = <&i2c3m0_pins>;
            status = "disabled";
        };

        i2c4: i2c@21130000 {
            compatible = "rockchip,rv1126b-i2c";
            reg = <0x21130000 0x1000>;
            interrupts = <GIC_SPI 52 IRQ_TYPE_LEVEL_HIGH>;
            #address-cells = <1>;
            #size-cells = <0>;
            clocks = <&cru CLK_I2C4>, <&cru PCLK_I2C4>;
            clock-names = "i2c", "pclk";
            pinctrl-names = "default";
            pinctrl-0 = <&i2c4m0_pins>;
            status = "disabled";
        };

        i2c5: i2c@21140000 {
            compatible = "rockchip,rv1126b-i2c";
            reg = <0x21140000 0x1000>;
            interrupts = <GIC_SPI 53 IRQ_TYPE_LEVEL_HIGH>;
            #address-cells = <1>;
            #size-cells = <0>;
            clocks = <&cru CLK_I2C5>, <&cru PCLK_I2C5>;
            clock-names = "i2c", "pclk";
            pinctrl-names = "default";
            pinctrl-0 = <&i2c5m0_pins>;
            status = "disabled";
        };



This article mainly describes how to configure I2C on the development board.

Configuring I2C can be divided into two steps:

* Define and register I2C devices.
* Define and register I2C drivers.

Let's take configuring GSL3680 as an example.

## Define and register I2C devices

When registering an I2C device, the `i2c_client` structure is required to describe the I2C device. However, in standard Linux, users only need to provide the corresponding I2C device information, and Linux will construct the `i2c_client` structure based on the information provided.

The I2C device information provided by the user is written to the DTS file in the form of nodes, as shown below:

```
kernel/arch/arm64/boot/dts/rockchip/rk3588-firefly-aioj-lvds-HSX101H40C.dts
&i2c2 {
    status = "okay";
    clock-frequency = <100000>;
    //i2c-scl-rising-time-ns = <800>;
    //i2c-scl-falling-time-ns = <100>;
    gslx680: gslx680@41 {
        compatible = "gslX680";
        reg = <0x41>;
        screen_max_x = <800>;
        screen_max_y = <1280>;
        touch-gpio = <&gpio0 RK_PC5 IRQ_TYPE_LEVEL_LOW>;
        reset-gpio = <&gpio0 RK_PC1 GPIO_ACTIVE_HIGH>;
        flip-x = <1>;
        flip-y = <0>;
        swap-xy = <0>;
        gsl,fw = <1>;
    };
};
```

## Define and register I2C drivers

### Define I2C drivers

Before defining the I2C driver, the user first defines the variables `of_device_id` and `i2c_device_id`.

`of_device_id` is used to call the device information defined in the DTS file in the driver, which is defined as follows:

```
static struct of_device_id gsl_ts_ids[] = {
    {.compatible = "gslX680"},
    {}
};
```

Define `i2c_device_id`:

```
static const struct i2c_device_id gsl_ts_id[] = {
    {GSLX680_I2C_NAME, 0},
    {}
};
 MODULE_DEVICE_TABLE(i2c, gsl_ts_id);
```

`i2c_driver` is shown as follows:

```
static struct i2c_driver gsl_ts_driver = {
    .driver = { .name = GSLX680_I2C_NAME,
    .owner = THIS_MODULE,
    .of_match_table = of_match_ptr(gsl_ts_ids),
    },
 #ifndef CONFIG_HAS_EARLYSUSPEND
    //.suspend  = gsl_ts_suspend,
    //.resume   = gsl_ts_resume,
 #endif
    .probe      = gsl_ts_probe,
    .remove     = gsl_ts_remove,
    .id_table   = gsl_ts_id,
};
```

Note: `id_table` indicates the devices supported by the driver.

### Register I2C drivers

Register the I2C driver with the `i2c_add_driver()` function.

```
i2c_add_driver(&gsl_ts_driver);
```

When `i2c_add_driver()` is called to register the I2C driver, the I2C device is traversed, and the `probe` function of the driver is called if the driver supports the device being traversed.

### Send and receive data through I2C

After registering the I2C driver, I2C communication can be carried out.

* Send information to slave machine:

```
int i2c_master_send(const struct i2c_client *client, const char *buf, int count)
{
    int ret;
    struct i2c_adapter *adap = client->adapter;
    struct i2c_msg msg;
    msg.addr = client->addr;
    msg.flags = client->flags & I2C_M_TEN;
    msg.len = count;
    msg.buf = (char *)buf;
    ret = i2c_transfer(adap, &msg, 1);
    /*
    + If everything went ok (i.e. 1 msg transmitted), return #bytes
    + transmitted, else error code.
    */
    return (ret == 1) ? count : ret;
}
```

* Read information to slave machine:

```
int i2c_master_recv(const struct i2c_client *client, char *buf, int count)
{
    struct i2c_adapter *adap = client->adapter;
    struct i2c_msg msg;
    int ret;
    msg.addr = client->addr;
    msg.flags = client->flags & I2C_M_TEN;
    msg.flags |= I2C_M_RD;
    msg.len = count;
    msg.buf = buf;
    ret = i2c_transfer(adap, &msg, 1);
    /*
    + If everything went ok (i.e. 1 msg received), return #bytes received,
    + else error code.
    */
    return (ret == 1) ? count : ret;
}
EXPORT_SYMBOL(i2c_master_recv);
```

## FAQs

### Q1: Communication failed. How do you debug 'log: "timeout, ipd: 0x00, state: 1"'?

**A1 :** Please check whether the hardware pull-up is powered.

### Q2: Call i2c_transfer and return -6?

**A2 :** A return value of -6 indicates an NACK error, that is, there is no response from the other device. This is generally a peripheral problem, which is common in the following situations:

* I2C address error, the solution is to measure the I2C waveform, confirm whether I2C device address is wrong.
* *I2C slave* device is not in normal working state, such as no power supply, wrong power on the timing sequence;
* Timing does not comply with the requirements of *I2C slave device* will also generate Nack signal.

### Q3: What happens when a peripheral requires a stop signal in the middle of a read sequence rather than a repeat start signal?

**A3 :** At this point, `i2c_transfer` needs to be called twice, and `I2C read` needs to be split into two times. The modification is as follows:

```
static int i2c_read_bytes(struct i2c_client *client, u8 cmd, u8 *data, u8 data_len) {
    struct i2c_msg msgs[2];
    int ret;
    u8 *buffer;
    buffer = kzalloc(data_len, GFP_KERNEL);
    if (!buffer)
        return -ENOMEM;;
    msgs[0].addr = client->addr;
    msgs[0].flags = client->flags;
    msgs[0].len = 1;
    msgs[0].buf = &cmd;
    ret = i2c_transfer(client->adapter, msgs, 1);
    if (ret < 0) {
        dev_err(&client->adapter->dev, "i2c read failed\n");
        kfree(buffer);
        return ret;
    }
    msgs[1].addr = client->addr;
    msgs[1].flags = client->flags | I2C_M_RD;
    msgs[1].len = data_len;
    msgs[1].buf = buffer;
    ret = i2c_transfer(client->adapter, &msgs[1], 1);
    if (ret < 0)
        dev_err(&client->adapter->dev, "i2c read failed\n");
    else
        memcpy(data, buffer, data_len);
    kfree(buffer);
    return ret;
}
```
