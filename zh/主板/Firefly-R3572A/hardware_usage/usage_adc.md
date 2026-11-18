# ADC 使用

## 简介 [step]

开发板上的 AD 接口为逐次逼近ADC (Successive Approximation Register)，单端 12 位，内核采用工业 I/O 子系统（IIO）来控制 ADC，该子系统主要为 AD 转换或者 DA 转换的传感器设计。下面以 SAR-ADC 为例子，介绍 Firefly-R3572A 的 ADC 基本使用方法。

## 硬件连接 [step]

ADC 信号通过 J1 双排排针引出，引脚定义如下：

| 排针序号 | 信号 | 电平 |
| --- | --- | --- |
| 15 | SARADC_VIN4 | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="注意" type="warn">

SARADC 输入电平为 1.8 V，输入电压不要超过该范围，否则可能损坏芯片。

</Callout>

## DTS 配置 [step]

### 配置 DTS 节点

SARADC 控制器节点由 SDK 在芯片级 DTS 中定义，板级 DTS 中需要申请使用的通道。以通道 4（SARADC_VIN4）为例，在 DTS 文件中添加 ADC 的资源描述：

```text
adc_demo: adc_demo {
    compatible = "firefly,rk3572-adc";
    status = "okay";
    io-channels = <&saradc 4>;
};
```

DTS 文件路径与 compatible 字符串以 Firefly-R3572A SDK 实际代码为准。

### 在驱动文件中匹配 DTS 节点

用户驱动可参考 Firefly adc demo 的写法。首先在驱动文件中定义 `of_device_id` 结构体数组：

```c
static const struct of_device_id firefly_adc_match[] = {
    { .compatible = "firefly,rk3572-adc" },
    {},
};
```

然后将该结构体数组填充到要使用 ADC 的 platform_driver 中：

```c
static struct platform_driver firefly_adc_driver = {
    .probe      = firefly_adc_probe,
    .remove     = firefly_adc_remove,
    .driver     = {
        .name   = "firefly_adc",
        .owner  = THIS_MODULE,
        .of_match_table = firefly_adc_match,
    },
};
```

接着在 firefly_adc_probe 中对 DTS 所添加的资源进行解析：

```c
static int firefly_adc_probe(struct platform_device *pdev)
{
     printk("firefly_adc_probe!\n");
     chan = iio_channel_get(&(pdev->dev), NULL);
     if (IS_ERR(chan))
     {
          chan = NULL;
          printk("%s() have not set adc chan\n", __FUNCTION__);
          return -1;
     }
     return 0;
}
```

## 驱动说明 [step]

### 获取 AD 通道

```c
struct iio_channel *chan;     //定义 IIO 通道结构体
chan = iio_channel_get(&pdev->dev, NULL);    //获取 IIO 通道结构体
```

**注意：** `iio_channel_get` 通过 probe 函数传进来的参数 pdev 获取 IIO 通道结构体，probe 函数如下：

```c
static int XXX_probe(struct platform_device *pdev);
```

### 读取 AD 采集到的原始数据

```c
int val, ret;
ret = iio_read_channel_raw(chan, &val);
```

调用 iio_read_channel_raw 函数读取 AD 采集的原始数据并存入 val 中。

### 计算采集到的电压

使用标准电压将 AD 转换的值转换为用户所需要的电压值。其计算公式如下：

```text
Vref / (2^n-1) = Vresult / raw
```

注意：

* Vref 为标准电压
* n 为 AD 转换的位数
* Vresult 为用户所需要的采集电压
* raw 为 AD 采集的原始数据

例如，标准电压为 1.8V，AD 采集位数为 12 位，AD 采集到的原始数据为 445，则：

```text
Vresult = (1800mv * 445) / 4095;
```

## 接口说明 [step]

```c
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

* 功能：获取 iio 通道描述
* 参数：
  * dev: 使用该通道的设备描述指针
  * consumer_channel: 该设备所使用的 IIO 通道描述指针

```c
void iio_channel_release(struct iio_channel *chan);
```

* 功能：释放 iio_channel_get 函数获取到的通道
* 参数：
  * chan：要被释放的通道描述指针

```c
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

* 功能：读取 chan 通道 AD 采集的原始数据。
* 参数：
  * chan：要读取的采集通道指针
  * val：存放读取结果的指针

## 调试方法 [step]

### 读取 ADC 值

系统启动后，可以通过 IIO 设备节点直接读取 ADC 采集的原始数据：

```bash
cat /sys/bus/iio/devices/iio\:device0/in_voltage4_raw
```

设备节点编号以系统实际生成为准，可通过 `ls /sys/bus/iio/devices/` 确认。

### 获取所有 ADC 值

有个便捷的方法可以查询到每个 SARADC 的值：

```bash
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```

## FAQs [step]

### 为何按上面的步骤申请 SARADC，会出现申请报错的情况？

驱动需要获取 ADC 通道来使用时，需要对驱动的加载时间进行控制，必须要在 saradc 初始化之后。saradc 是使用 module_platform_driver() 进行平台设备驱动注册，最终调用的是 module_init()。所以用户的驱动加载函数只需使用比 module_init() 优先级低的，例如：late_initcall()，就能保证驱动的加载的时间比 saradc 初始化时间晚，可避免出错。
