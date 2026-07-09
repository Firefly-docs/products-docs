# ADC 使用

## 前言

AIO-3288J 开发板上的 AD 接口分为：高速 ADC 流接口 (High-speed ADC Stream Interface)、温度传感器 (Temperature Sensor)、逐次逼近ADC (Successive Approximation Register)。本文主要介绍 ADC 的基本配置方法。

内核采用工业 I/O 子系统来控制 ADC，该子系统主要为 AD 转换或者 DA 转换的传感器设计。其相关数据结构以及配置方法如下：

## 数据结构

* iio_channel 结构体

```c
struct iio_channel {
    struct iio_dev *indio_dev; //工业 I/O设备
    const struct iio_chan_spec *channel; //I/O通道
    void *data;
};
```

* iio_dev 结构体

该结构体主要用于描述 IO 口所属的设备，其具体定义如下：

```c
struct iio_dev {
    int             id;
    int             modes;
    int             currentmode;
    struct device           dev;
    struct iio_event_interface  *event_interface;
    struct iio_buffer       *buffer;
    struct list_head        buffer_list;
    int             scan_bytes;
    struct mutex            mlock;
    const unsigned long     *available_scan_masks;
    unsigned            masklength;
    const unsigned long     *active_scan_mask;
    bool                scan_timestamp;
    unsigned            scan_index_timestamp;
    struct iio_trigger      *trig;
    struct iio_poll_func        *pollfunc;
    struct iio_chan_spec const  *channels;
    int             num_channels;
    struct list_head        channel_attr_list;
    struct attribute_group      chan_attr_group;
    const char          *name;
    const struct iio_info       *info;
    struct mutex            info_exist_lock;
    const struct iio_buffer_setup_ops   *setup_ops;
    struct cdev         chrdev;
    #define IIO_MAX_GROUPS 6
    const struct attribute_group    *groups[IIO_MAX_GROUPS + 1];
    int             groupcounter;
    unsigned long           flags;
    #if defined(CONFIG_DEBUG_FS)
    struct dentry           *debugfs_dentry;
    unsigned            cached_reg_addr;
    #endif
};
```

* iio_chan_spec结构体

```c
struct iio_chan_spec {
    enum iio_chan_type  type; //描述通道类型
    int  channel; //通道号
    int  channel2; //通道号
    unsigned long  address; //通道地址
    int  scan_index;
    struct {
        char    sign;
        u8  realbits;
        u8  storagebits;
        u8  shift;
        enum iio_endian endianness;
    } scan_type;
    long            info_mask;
    long            info_mask_separate;
    long            info_mask_shared_by_type;
    long            event_mask;
    const struct iio_chan_spec_ext_info *ext_info;
    const char      *extend_name;
    const char      *datasheet_name;
    unsigned        modified:1;
    unsigned        indexed:1;
    unsigned        output:1;
    unsigned        differential:1;
};
```

## 配置步骤

### 配置DTS节点

Firefly ADC 的 DTS 节点在 `kernel/arch/arm/boot/dts/rk3288.dtsi` 文件中定义，如下所示：

```dts
adc: adc@ff100000 {
     compatible = "rockchip,saradc";
     reg = <0xff100000 0x100>;
     interrupts = <GIC_SPI 36 IRQ_TYPE_LEVEL_HIGH>;
     #io-channel-cells = <1>;
     io-channel-ranges;rockchip,adc-vref = <1800>;
     clock-frequency = <1000000>;
     clocks = <&clk_saradc>, <&clk_gates7 1>;
     clock-names = "saradc", "pclk_saradc";
     status = "disabled";
 };
```

用户只需在 firefly-rk3288-aio-3288j.dts 文件中添加通道定义，并将其 status 改为 "okay" 即可：

```dts
&adc {
    status = "okay";
    adc_test{
        compatible = "rockchip,adc_test";
        io-channels = <&adc 0>;
    };
};
```

### 在驱动文件中匹配 DTS 节点

在驱动文件中定义 of_device_id 结构体数组：

```c
static const struct of_device_id of_XXX_match[] = {
    { .compatible = "rockchip,adc_test" },
    { /* Sentinel */ }
};
```

将该结构体数组填充到要使用 ADC 的 platform_driver 中。

```c
static struct platform_driver XXX_driver = {
    .probe      = ...,
    .remove     = ...,
    .driver     = {
        .name   = "..",
        .owner  = THIS_MODULE,
    #ifdef CONFIG_OF
        .of_match_table = of_XXX_match,
    #endif
    },  
};
```

### 获取 AD 通道

```c
struct iio_channel *chan; //定义 IIO 通道结构体
chan = iio_channel_get(&pdev->dev, NULL); //获取 IIO 通道结构体
```

注：iio_channel_get 通过 probe 函数传进来的参数 pdev 获取 IIO 通道结构体，probe 函数如下：

```c
static int XXX_probe(struct platform_device *pdev);
```

### 读取 AD 采集到的原始数据

```c
int val,ret;
ret = iio_read_channel_raw(chan, &val);
```

调用 iio_read_channel_raw 函数读取 AD 采集的原始数据并存入 val 中。

### 计算采集到的电压

使用标准电压将 AD 转换的值转换为用户所需要的电压值。其计算公式如下：

```c
Vref / (2^n-1) = Vresult / raw
```

注：

* Vref 为标准电压
* n 为 AD 转换的位数
* Vresult 为用户所需要的采集电压
* raw 为 AD 采集的原始数据

例如，标准电压为 1.8V，AD 采集位数为 10 位，AD 采集到的原始数据为 568，则：

```c
Vresult = (1800mv * 568) / 1023;
```

## ADC 常用函数接口

```c
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

* 功能：获取 iio 通道描述
* 参数：
    + dev: 使用该通道的设备描述指针
    + consumer_channel: 该设备所使用的 IIO 通道描述指针

```c
void iio_channel_release(struct iio_channel *chan);
```

* 功能：释放 iio_channel_get 函数获取到的通道
* 参数：
    + chan：要被释放的通道描述指针

```c
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

* 功能：读取 chan 通道 AD 采集的原始数据
* 参数：
    + chan：要读取的采集通道指针
    + val：存放读取结果的指针