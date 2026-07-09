# I2C Use

## Introduction

There are 6 on-chip I2C controllers on the AIO-3288J development board. This paper describes how to configure the I2C on the development board.

There are two major steps to configure the I2C:  

* Define and register the I2C device
* Define and register the I2C driver

Here is an example of configuring the lt8641ex.  

## Define and register the I2C device

The structure i2c_client is required to describe the I2C device when registering the I2C device. However, the user only needs to provide the appropriate I2C device information in standard Linux, and Linux will construct the i2c_client structure based on the information provided.

The I2C device information provided by the user is written to the dts file as a node, and it is shown as follows:

```dts
&i2c1 {
    status = "okay";
    lt8641ex@3f {
        compatible = "firefly,lt8641ex";
        gpio-sw = <&gpio7 GPIO_B2 GPIO_ACTIVE_LOW>;
        reg = <0x3f>;
    };
    rtc@51 {
        compatible = "nxp,pcf8563";
        reg = <0x51>;
    };
};
```

## Define and register the I2C driver

### Define the I2C driver

Before defining I2C drivers, the user first defines the variables of_device_id and i2c_device_id.  
The of_device_id is used to call the device information defined in the dts file in the driver, it is defined as follows:

```c
static const struct of_device_id of_rk_lt8641ex_match[]={
    { .compatible = "firefly,lt8641ex" },
    { /* Sentinel */ }
};
```

Define the variable i2c_device_id：

```c
static const struct i2c_device_id lt8641ex_id[] = {
    { lt8641ex, 0 },
    { }
};
MODULE_DEVICE_TABLE(i2c, lt8641ex_id);
```

i2c _ driver is shown as follows:

```c
static struct i2c_driver lt8641ex_device_driver = {
    .driver     = {
        .name   = "lt8641ex",
        .owner  = THIS_MODULE,
        .of_match_table = of_rk_lt8641ex_match,
     },  
    .probe      = lt8641ex_probe,
    .remove     = lt8641ex_remove,
    .suspend        = lt8641ex_suspend,
    .resume         = lt8641ex_resume,
    .id_table       = lt8641ex_id,
};
```

Note: the variable id_table indicates the device supported by the driver.

### Register the I2C driver

Register the I2C driver with the i2c_add_driver function.

```c
i2c_add_driver(&lt8641ex_device_driver);
```

The I2C device is traversed when the i2c_add_driver is called to register the I2C driver, and the probe function of the driver is called if the driver supports the device to which it is traversing.

### Send and receive the data via I2C

Once the I2 C driver has been registered, the I2C communication can be carried out.

* Send the message to slave

```c
static int i2c_master_reg8_send(const struct i2c_client *client, const char reg, const char *buf, int count, int scl_rate) {
    struct i2c_adapter *adap=client->adapter;
    struct i2c_msg msg;
    int ret;
    char *tx_buf = (char *)kzalloc(count + 1, GFP_KERNEL);
    if(!tx_buf)
        return -ENOMEM;
    tx_buf[0] = reg;
    memcpy(tx_buf+1, buf, count);  
    msg.addr = client->addr;
    msg.flags = client->flags;
    msg.len = count + 1;
    msg.buf = (char *)tx_buf;
    msg.scl_rate = scl_rate;
    ret = i2c_transfer(adap, &msg, 1);
    kfree(tx_buf);
    return (ret == 1) ? count : ret;
}
```

* Read the message from slave

```c
static int i2c_master_reg8_recv(const struct i2c_client *client, const char reg, char *buf, int count, int scl_rate) {
    struct i2c_adapter *adap=client->adapter;
    struct i2c_msg msgs[2];
    int ret;
    char reg_buf = reg; 
    msgs[0].addr = client->addr;
    msgs[0].flags = client->flags;
    msgs[0].len = 1;
    msgs[0].buf = &reg_buf;
    msgs[0].scl_rate = scl_rate; 
    msgs[1].addr = client->addr;
    msgs[1].flags = client->flags | I2C_M_RD;msgs[1].len = count;
    msgs[1].buf = (char *)buf;
    msgs[1].scl_rate = scl_rate; 
    ret = i2c_transfer(adap, msgs, 2);  
    return (ret == 2)? count : ret;
}
```

Note:

The msgs[0] is the message to be sent to the slave and tells the slave the message that the slave needs to read from the salve.  

The msgs[1] is the information that the host reads from the slave.

At this point, the host can communicate with the salve using the functions i2c_master_reg8_send and i2c_master_reg8_recv.

* Actual communications example

For example, the host communicats with LT8641EX, the host sends the information to LT8641EX,  the use channel of LT8641EX  is set to 1:

```c
int channel=1;
i2c_master_reg8_send(g_lt8641ex->client, 0x00, &channel,1, 100000);
```

Note: the address of the channel register is 0x00.

The host reads the currently used channel from the slave LT8641EX:

```c
u8 ch = 0xfe;
i2c_master_reg8_recv(g_lt8641ex->client, 0x00, &ch,1, 100000);
```

Note: ch is used to save the read information.