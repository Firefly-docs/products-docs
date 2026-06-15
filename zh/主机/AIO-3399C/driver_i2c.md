# I2C 使用

## 简介

AIO-3399C 开发板上有 9 个片上 I2C 控制器，各个 I2C 的使用情况如下表：

![](../../../rk3399_img/AIO-3399C/i2c_list.jpg)

本文主要描述如何在该开发板上配置 I2C。

配置 I2C 可分为两大步骤：

*    定义和注册 I2C 设备
*    定义和注册 I2C 驱动

下面以配置 GSL3680 为例。

## 定义和注册 I2C 设备

在注册 I2C 设备时，需要结构体 `i2c_client` 来描述 I2C 设备。然而在标准 Linux 中，用户只需要提供相应的 I2C 设备信息，Linux 就会根据所提供的信息构造 `i2c_client` 结构体。

用户所提供的 I2C 设备信息以节点的形式写到 DTS 文件中，如下所示：

```
kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-edp.dts
&i2c4 {
    status = "okay";
    gsl3680: gsl3680@41 {
              compatible = "gslX680";
              reg = <0x41>;
              screen_max_x = <1536>;
              screen_max_y = <2048>;
              touch-gpio = <&gpio1 20 IRQ_TYPE_LEVEL_LOW>;
              reset-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;
      };
};
```

## 定义和注册 I2C 驱动

### 定义 I2C 驱动

在定义 I2C 驱动之前，用户首先要定义变量 `of_device_id` 和 `i2c_device_id`。

`of_device_id` 用于在驱动中调用 DTS 文件中定义的设备信息，其定义如下所示：

```
 static struct of_device_id gsl_ts_ids[] = {
   {.compatible = "gslX680"},
   {}
 };
```

定义变量 `i2c_device_id`：

```
 static const struct i2c_device_id gsl_ts_id[] = {
    {GSLX680_I2C_NAME, 0},
    {}
 };
 MODULE_DEVICE_TABLE(i2c, gsl_ts_id);
```

`i2c_driver` 如下所示：

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

注：变量 `id_table` 指示该驱动所支持的设备。

### 注册 I2C 驱动

使用 `i2c_add_driver` 函数注册 I2C 驱动。

```
i2c_add_driver(&gsl_ts_driver);
```

在调用 `i2c_add_driver` 注册 I2C 驱动时，会遍历 I2C 设备，如果该驱动支持所遍历到的设备，则会调用该驱动的 `probe` 函数。

### 通过 I2C 收发数据

在注册好 I2C 驱动后，即可进行 I2C 通讯。

*    向从机发送信息：

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
      * If everything went ok (i.e. 1 msg transmitted), return #bytes
      * transmitted, else error code.
      */
     return (ret == 1) ? count : ret;
 }
```

*    向从机读取信息：

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
      * If everything went ok (i.e. 1 msg received), return #bytes received,
      * else error code.
      */
     return (ret == 1) ? count : ret;
 }
 EXPORT_SYMBOL(i2c_master_recv);
```

## FAQs

### Q1: 通信失败，出现这种 log: "timeout, ipd: 0x00, state: 1" 该如何调试？

A1: 请检查硬件上拉是否给电。

### Q2: 调用 i2c_transfer 返回值为 -6？

A2: 返回值为 -6 表示为 NACK 错误，即对方设备无应答响应，这种情况一般为外设的问题，常见的有以下几种情况：

*    I2C 地址错误，解决方法是测量 I2C 波形，确认是否 I2C 设备地址错误；
*    I2C slave 设备不处于正常工作状态，比如未给电，错误的上电时序等；
*    时序不符合 I2C slave 设备所要求也会产生 Nack 信号。

### Q3: 当外设对于读时序要求中间是 stop 信号不是 repeat start 信号的时候，该如何处理？

A3: 这时需要调用两次 i2c_transfer, I2C read 拆分成两次，修改如下：

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
