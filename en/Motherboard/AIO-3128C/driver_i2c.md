# I2C Use

## Introduction
There are 4 on-chip I2C controllers in AIO-3128C development board, whose configuration and usage will be introduced below.  
In order to config I2C, there are mainly two steps:  

* Define and register the I2C device.
* Define and register the I2C driver.

Let's take the lt8641ex for example.

## Define and Register the I2C Device

You need an i2c_client struct to describe the i2c device. However this struct do not need to defined directly. The only thing you need to do is providing the device information and the kernel will build one from it.
You can write the information of the I2C device as a node in the dts file, which is shown below:
```
&i2c0 {
    status = "okay";
    lt8641ex@3f {
        compatible = "firefly,lt8641ex";
        gpio-sw = <&gpio7 GPIO_B2 GPIO_ACTIVE_LOW>;
        reg = <0x3f>;
    };
};
```

## Define and Register the I2C Driver

### Define the I2C Driver

Before defining i2_driver, you need to define of_device_id and i2c_device_id first.
of_device_id is defined to match the corresponding node in the dts file:
```
static const struct of_device_id of_rk_lt8641ex_match[] = { { .compatible = "firefly,lt8641ex"  }, { /* Sentinel */ } ;
```

define i2c_device_id:
```
static const struct i2c_device_id lt8641ex_id[] = { { lt8641ex, 0 }, { } };MODULE_DEVICE_TABLE(i2c, lt8641ex_id);
```

Define i2c_driver as following:
```
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
    .id_table       = lt8641ex_id,};
```
NOTE: id_table shows which device the driver support.

### Register the I2C Driver

Use i2c_add_driver to register I2C  driver.
```
i2c_add_driver(&lt8641ex_device_driver);
```

During the call to i2c_add_driver to register I2C driver, all the I2C devices will be traversed. Once matched, the probe function of the driver will be executed.

### Send and receive the data via I2C

Once the I2 C driver has been registered, the I2C communication can be carried out.

* Send the message to slave
```
static int i2c_master_reg8_send(const struct i2c_client *client, const char reg, const char *buf, int count, int scl_rate) {
    struct i2c_adapter *adap=client->adapter;
    struct i2c_msg msg;int ret;
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
```
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
Note:The msgs[0] is the message to be sent to the slave and tells the slave the message that the slave needs to read from the salve.The msgs[1] is the information that the host reads from the slave.

At this point, the host can communicate with the salve using the functions i2c_master_reg8_send and i2c_master_reg8_recv.  

* Actual communications example
For example, the host communicats with LT8641EX, the host sends the information to LT8641EX, the use channel of LT8641EX is set to 1:
```
int channel=1;i2c_master_reg8_send(g_lt8641ex->client, 0x00, &channel,1, 100000);
```
Note: the address of the channel register is 0x00. The host reads the currently used channel from the slave LT8641EX:
```
u8 ch = 0xfe;i2c_master_reg8_recv(g_lt8641ex->client, 0x00, &ch,1, 100000);
```
Note: ch is used to save the read information.