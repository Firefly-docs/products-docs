# I2C 使用

## 前言

AIO-3288C 开发板上有 6 个片上 I2C 控制器。本文主要描述如何在该开发板上配置 I2C。

- 配置 I2C 可分为两大步骤：
    + 定义和注册 I2C 设备
    + 定义和注册 I2C 驱动

下面以配置 lt8641ex 为例。

## 定义和注册 I2C 设备

在注册I2C设备时，需要结构体 i2c_client 来描述 I2C 设备。然而在标准Linux中，用户只需要提供相应的 I2C 设备信息，Linux就会根据所提供的信息构造 i2c_client 结构体。

用户所提供的 I2C 设备信息以节点的形式写到 dts 文件中，如下所示：

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

## 定义和注册 I2C 驱动

### 定义 I2C 驱动

在定义 I2C 驱动之前，用户首先要定义变量 of_device_id 和 i2c_device_id 。
of_device_id 用于在驱动中调用dts文件中定义的设备信息，其定义如下所示：

```c
static const struct of_device_id of_rk_lt8641ex_match[] = {
    { .compatible = "firefly,lt8641ex" },
    { /* Sentinel */ }
};
```

定义变量 i2c_device_id：

```c
static const struct i2c_device_id lt8641ex_id[] = {
    { lt8641ex, 0 },
    { }
};
MODULE_DEVICE_TABLE(i2c, lt8641ex_id);
```

i2c_driver 如下所示：

```c
static struct i2c_driver lt8641ex_device_driver = {
    .driver     = {
        .name   = "lt8641ex",
        .owner  = THIS_MODULE,.of_match_table = of_rk_lt8641ex_match,
    },  
    .probe          = lt8641ex_probe,
    .remove         = lt8641ex_remove,
    .suspend        = lt8641ex_suspend,
    .resume         = lt8641ex_resume,
    .id_table       = lt8641ex_id,
};
```

注：变量id_table指示该驱动所支持的设备。

### 注册 I2C 驱动

使用i2c_add_driver函数注册 I2C 驱动。

```c
i2c_add_driver(&lt8641ex_device_driver);
```

在调用 i2c_add_driver 注册 I2C 驱动时，会遍历 I2C 设备，如果该驱动支持所遍历到的设备，则会调用该驱动的 probe 函数。

### 通过 I2C 收发数据

在注册好 I2 C 驱动后，即可进行 I2C 通讯。

* 向从机发送信息

```c
static int i2c_master_reg8_send(const struct i2c_client *client, const char reg, const char *buf, int count, int scl_rate){
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

* 向从机读取信息

```c
static int i2c_master_reg8_recv(const struct i2c_client *client, const char reg, char *buf, int count, int scl_rate){
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
    msgs[1].flags = client->flags | I2C_M_RD;msgs[1].len = count;msgs[1].buf = (char *)buf;
    msgs[1].scl_rate = scl_rate;
    ret = i2c_transfer(adap, msgs, 2);
    return (ret == 2)? count : ret;
}
```

注：msgs[0] 是要向从机发送的信息，告诉从机主机要读取信息。msgs[1] 是主机向从机读取到的信息。

至此，主机可以使用函数 i2c_master_reg8_send 和 i2c_master_reg8_recv 和从机进行通讯。

* 实际通讯示例

例如主机和 LT8641EX 通讯，主机向 LT8641EX 发送信息，设置 LT8641EX 使用通道 1：

```c
int channel=1;
i2c_master_reg8_send(g_lt8641ex->client, 0x00, &channel,1, 100000);
```

注：通道寄存器的地址为0x00。

主机向从机 LT8641EX 读取当前使用的通道：

```c
u8 ch = 0xfe;
i2c_master_reg8_recv(g_lt8641ex->client, 0x00, &ch,1, 100000);
```

注：ch用于保存读取到的信息。