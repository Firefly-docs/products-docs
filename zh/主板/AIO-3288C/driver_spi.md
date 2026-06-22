# SPI 使用

SPI 是一种高速的，全双工，同步串行通信接口，用于连接微控制器、传感器、存储设备等，本文以指纹识别模块为例简单介绍 SPI 使用。

## SPI 工作方式

SPI 以主从方式工作，这种模式通常有一个主设备和一个或多个从设备，需要至少 4 根线，分别是：

```c
CS          片选信号
SCLK        时钟信号
MOSI        主设备数据输出、从设备数据输入
MISO        主设备数据输入，从设备数据输出
```

Linux 内核用 CPOL 和 CPHA 的组合来表示当前 SPI 的四种工作模式：

```c
CPOL＝0，CPHA＝0       SPI_MODE_0
CPOL＝0，CPHA＝1       SPI_MODE_1
CPOL＝1，CPHA＝0       SPI_MODE_2
CPOL＝1，CPHA＝1       SPI_MODE_3
```

CPOL：表示时钟信号的初始电平的状态，０ 为低电平，１ 为高电平。  

CPHA：表示在哪个时钟沿采样，０ 为第一个时钟沿采样，１ 为第二个时钟沿采样。  

SPI的四种工作模式波形图如下：

![](../../../rk3288_img/spi.jpg)

## 在内核添加自己的驱动文件

在内核源码目录 `kernel/drivers/spi/` 中创建新的驱动文件，如：spi-rockchip-firefly.c 在驱动文件所在目录下的 Kconfig 文件添加对应的驱动文件配置，如：

```c
@@ -525,6 +525,10 @@ config SPI_ROCKCHIP_TEST
       bool "ROCKCHIP spi test code"
       depends on SPI_ROCKCHIP

+config SPI_ROCKCHIP_FIREFLY
+       bool "ROCKCHIP spi firefly code"
+       depends on SPI_ROCKCHIP
+
#
# There are lots of SPI device types, with sensors and memory
# being probably the most widely used ones.
```

在驱动文件所在目录下的 Makefile 文件添加对应的驱动文件名，如：

```c
+obj-$(CONFIG_SPI_ROCKCHIP_FIREFLY) += spi-rockchip-firefly.o
```

用 `make menuconfig` 在内核选项中选中所添加的驱动文件，如：

```c
There is no help available for this option.
│ Symbol: SPI_ROCKCHIP_FIREFLY [=y]
│ Type  : boolean
│ Prompt: ROCKCHIP spi firefly code
│   Location:
│     -> Device Drivers
│       -> SPI support (SPI [=y])
│         -> ROCKCHIP SPI controller core support (SPI_ROCKCHIP_CORE [=y])
│           -> ROCKCHIP SPI interface driver (SPI_ROCKCHIP [=y])
│   Defined at drivers/spi/Kconfig:528  
│   Depends on: SPI [=y] && SPI_MASTER [=y] && SPI_ROCKCHIP [=y]
```

## 定义和注册 SPI 设备

在 DTS 中添加 SPI 驱动结点描述，如下所示：`kernel/arch/arm/boot/dts/firefly-rk3288.dts`

```dts
&spi0 {
    status = "okay";
    max-freq = <24000000>;
    spidev@00 {
        compatible = "rockchip,spi_firefly";
        reg = <0x00>;
        spi-max-frequency = <14000000>;
        spi-cpha = <1>;
        //spi-cpol = <1>;
    };
};
```

* status：如果要启用 SPI，则设为 okay，如不启用，设为 disable。  
* spidev@00：由于本例子使用的是 SPI0，且使用 CS0，故此处设为 00，如果使用 CS1，则设为 01。  
* compatible：这里的属性必须与驱动中的结构体 of_device_id 中的成员 compatible 保持一致。  
* reg：此处与 spidev@00 保持一致，本例设为：0x00；
* spi-max-frequency：此处设置 spi 使用的最高频率。
* spi-cpha，spi-cpol：SPI 的工作模式在此设置，本例所用的模块 SPI 工作模式为 SPI_MODE_1，故设：spi-cpha = <1>，如果您所用设备工作模式为 SPI_MODE0，则需在此把这两个注释掉，如果用 SPI_MODE3，则设：spi-cpha = <1>;spi-cpol = <1>。

## 定义和注册 SPI 驱动

### 定义 SPI 驱动

在定义 SPI 驱动之前，用户首先要定义变量 of_device_id 。 of_device_id 用于在驱动中调用 dts 文件中定义的设备信息，其定义如下所示：

```c
static const struct of_device_id spidev_dt_ids[] = {
    { .compatible = "rockchip,spi_firefly" },
    {},
};
```

此处的compatible与DTS文件中的保持一致。

定义spi_driver如下所示：

```c
static struct spi_driver spidev_spi_driver = {
    .driver = {
        .name =         "silead_fp",
        .owner =        THIS_MODULE,
        .of_match_table = of_match_ptr(spidev_dt_ids),
     },
    .probe =        spi_gsl_probe,  
    .remove =       spi_gsl_remove,
};
```

### 注册SPI驱动

在初始化函数static int __init spidev_init(void)中创建一个字符设备：

```c
alloc_chrdev_region(&devno, 0,255, "sileadfp");
```

向内核添加该设备：

```c
spidev_major = MAJOR(devno);
cdev_init(&spicdev, &spidev_fops);
spicdev.owner = THIS_MODULE;
status = cdev_add(&spicdev,MKDEV(spidev_major, 0),N_SPI_MINORS);
```

创建设备类：

```cj
class_create(THIS_MODULE, "spidev");
```

向内核注册SPI驱动：

```c
spi_register_driver(&spidev_spi_driver);
```

如果内核启动时匹配成功，则调用该驱动的probe函数。 probe函数如下所示：

```c
static int spi_gsl_probe(struct spi_device *spi)
{
    struct spidev_data  *spidev;
    int                 status;
    unsigned long       minor;
    struct gsl_fp_data  *fp_data;
    printk("===============spi_gsl_probe ==============\n");
    if(!spi)
        return -ENOMEM;

    /* Allocate driver data */
    spidev = kzalloc(sizeof(*spidev), GFP_KERNEL);
    if (!spidev)
        return -ENOMEM;

    /* Initialize the driver data */
    spidev->spi = spi;
    spin_lock_init(&spidev->spi_lock);//初始化自旋锁
    mutex_init(&spidev->buf_lock);//初始化互斥锁
    INIT_LIST_HEAD(&spidev->device_entry);//初始化设备链表

    //init fp_data
    fp_data = kzalloc(sizeof(struct gsl_fp_data), GFP_KERNEL);
    if(fp_data == NULL){
        status = -ENOMEM;
        return status;
    }
    //set fp_data struct value
    fp_data->spidev = spidev;

    mutex_lock(&device_list_lock);//上互斥锁
    minor = find_first_zero_bit(minors, N_SPI_MINORS);//在内存区中查找第一个值为0的位
    if (minor < N_SPI_MINORS) {
        struct device *dev;
        spidev->devt = MKDEV(spidev_major, minor);
        dev = device_create(spidev_class, &spi->dev, spidev->devt, spidev, "silead_fp_dev");创建/dev/下设备结点
        status = IS_ERR(dev) ? PTR_ERR(dev) : 0;
    } else {
        dev_dbg(&spi->dev, "no minor number available!\n");
        status = -ENODEV;
    }
    if (status == 0) {
        set_bit(minor, minors);
        list_add(&spidev->device_entry, &device_list);//添加进设备链表
    }
    mutex_unlock(&device_list_lock);//解互斥锁
    if (status == 0)
        spi_set_drvdata(spi, spidev);
    else
        kfree(spidev);
        printk("%s:name=%s,bus_num=%d,cs=%d,mode=%d,speed=%d\n",__func__,spi->modalias, spi->master->bus_num, spi->chip_select, spi->mode,
spi->max_speed_hz);//打印SPI信息
    return status;
}
```

如果注册 SPI 驱动成功，你可以在 `/dev/` 目录下面看你到注册的驱动名称，可以在 `sys/class/` 下面看到你注册的驱动设备类。

## SPI 读写数据过程

### SPI 写数据

```c
static ssize_t spidev_write(struct file *filp, const char __user *buf, size_t count, loff_t *f_pos)
{
    struct spidev_data      *spidev;
    ssize_t                 status = 0;
    unsigned long           missing;
    if (count > bufsiz)
        return -EMSGSIZE;
    spidev = filp->private_data;
    mutex_lock(&spidev->buf_lock);
    missing = copy_from_user(spidev->buffer, buf, count);//把数据从用户空间传到内核空间
    if (missing == 0) {
        status = spidev_sync_write(spidev, count);//调用写同步函数
    } else
        status = -EFAULT;
    mutex_unlock(&spidev->buf_lock);
    return status;
}
```

写同步函数：

```c
spidev_sync_write(struct spidev_data *spidev, size_t len)
{
    struct spi_transfer t = {
        .tx_buf         = spidev->buffer,//发送缓冲区
        .len            = len,//发送数据长度
    };
    struct spi_message      m;
    spi_message_init(&m);//初始化spi_message
    spi_message_add_tail(&t, &m);//将新的spi_transfer添加到spi_message队列尾部
    return spidev_sync(spidev, &m);//同步读写
}
```

### SPI 读数据

在本例所用的模块中，读数据的过程为：  

* 主机向模块写寄存器的地址及读的指令（如：地址为 0xf0，读指令为 0x00)
* 模块收到读的指令后，数据以页的形式发送
* 主机设置读的模式
* 主机读取一页数据并存储

```c
static ssize_t
spidev_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
{
    struct spidev_data  *spidev;
    int         status = 0;
    int         i = 0;

    spidev = filp->private_data;
    mutex_lock(&spidev->buf_lock);
    gsl_fp_write(spidev, 0x00, 0xf0);//读之前先向模块写读的指令及寄存器地址
    while(1){
        for(i=0;i <= 110*118/128/read_pages;i++){
            status = gsl_fp_getOneFrame(spidev,0x00);//读1页数据
        }
        pos = 0;
        break;
    }
    if(status > 0){
        printk("gsl read data success!!!\n");
    }else{
        printk("gsl read data failed!!!");
    }
    mutex_unlock(&spidev->buf_lock);
    return status;
}
```

```c
static inline unsigned int
gsl_fp_getOneFrame(struct spidev_data *spidev,unsigned char reg_8b)
{
    int status,i;
    unsigned char buf_d[128*1+3] = {0x00,0x00};
    struct spi_transfer t;
    t.tx_buf = buf_d;
    t.rx_buf = buf_d;
    t.len = 131;
    status = gsl_spidev_sync_read(spidev, &t);

    if (status > 0){

        for(i=0;i<128*read_pages;i++)
            kmalloc_area[pos++] = buf_d[i+3];
    }

    return status;
}
```

```c
static inline ssize_t
gsl_spidev_sync_read(struct spidev_data *spidev,struct spi_transfer *t)
{
    struct spi_message m;
    spi_message_init(&m);
    t->bits_per_word = 8;//每次读的数据长度为8位
    t->delay_usecs = 1;//每次读完延时
    t->speed_hz = 14*1000*1000;//读的速率
    t->cs_change = 1;//CS引脚电平变化

    spi_message_add_tail(t, &m);
    return spidev_sync(spidev, &m);
}
```

注：Firefly的SPI驱动是Linux下通用的驱动，可以参考源码：`kernel/drivers/spi/spidev.c`