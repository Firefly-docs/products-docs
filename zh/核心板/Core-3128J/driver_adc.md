# ADC 使用

## 前言

Firefly-RK3128 开发板有一个 3 通道（0/1/2)、10 比特精度的 SAR ADC (Successive Approximation Register，逐次逼近寄存器)，其中：  

* ADCIN0: 在扩展板中引出
* ADCIN1: 内部作 Recovery 键检测
* ADCIN2: 在扩展板中引出


本文主要介绍 ADC 的基本使用方法。

## 配置DTS节点

Firefly-RK3128 ADC 的 DTS 节点在 kernel/arch/arm/boot/dts/rk312x.dtsi 文件中定义，如下所示：

```
adc: adc@2006c000 {compatible = "rockchip,saradc";
    reg = <0x2006c000 0x100>;
    interrupts = <GIC_SPI 17 IRQ_TYPE_LEVEL_HIGH>;
    #io-channel-cells = <1>;
    io-channel-ranges;
    rockchip,adc-vref = <1800>;
    clock-frequency = <1000000>;
    clocks = <&clk_saradc>, <&clk_gates7 14>;
    clock-names = "saradc", "pclk_saradc";
    status = "disabled";
};
```

用户只需在 rk3128-fireprime.dts 文件中添加通道定义，并将其 status 改为 "okay" 即可：

```
&adc {
    status = "okay";
    adc_test {status = "okay";
    compatible = "rk-adc-test";
    io-channels = <&adc 0>;
};
```

此处添加一个测试设备 adc_test，为下面的驱动测试例程所使用。 

ADC 的驱动源码为 drivers/iio/adc/rockchip_adc.c

## 获取 ADC 通道

```
struct iio_channel *chan;chan = iio_channel_get(&pdev->dev, "adc0");
```

adc0 为通道名称，可用的通道列表在 rockchip_adc.c 中定义：

```
static const struct iio_chan_spec rk_adc_iio_channels[] = {
	ADC_CHANNEL(0, "adc0"),ADC_CHANNEL(1, "adc1"),ADC_CHANNEL(2, "adc2"),ADC_CHANNEL(6, "adc6"),  
};
```

## 读取 AD 采集到的原始数据

```
int val, ret;ret = iio_read_channel_raw(chan, &val);
```

调用 iio_read_channel_raw 函数读取 ADC 采集的原始数据并存入 val 中。

## 计算采集到的电压

使用标准电压将 AD 转换的值转换为用户所需要的电压值。其计算公式如下：
```
Vref / (2^n-1) = Vresult / raw
```

注：
Vref 为标准电压
n 为 AD 转换的位数
Vresult 为用户所需要的采集电压
raw 为 AD 采集的原始数据
例如，标准电压为 1.8V，AD 采集位数为 10 位，AD 采集到的原始数据为 568，则：

```
Vresult = (1800mv * 568) / 1023;
```

## 驱动测试例程

以下为完整的读取 ADC 的驱动例程：

```
#include <linux/module.h>
#include <linux/err.h>
#include <linux/platform_device.h>
#include <linux/types.h>
#include <linux/adc.h>
#include <linux/string.h>
#include <linux/iio/iio.h>
#include <linux/iio/machine.h>
#include <linux/iio/driver.h>
#include <linux/iio/consumer.h> 
struct iio_channel *adc_test_channel; 
static ssize_t show_measure(struct device *dev, struct device_attribute *attr,   char *buf){
    int val, ret;
    size_t count = 0; 
    ret = iio_read_channel_raw(adc_test_channel, &val);
    if (ret < 0) {
        count += sprintf(&buf[count], "read channel() error: %d\n", ret);
    } 
    else 
    {
        count += sprintf(&buf[count], "read channel(): %d\n", val);
    }
    return count;
} 

static struct device_attribute measure_attr =__ATTR(measure, S_IRUGO, show_measure, NULL); 
static int rk_adc_test_probe(struct platform_device *pdev)
{
    struct iio_channel *channels;channels = iio_channel_get_all(&pdev->dev);
    if (IS_ERR(channels)) {
        pr_err("get adc channels fails\n");
        goto err;
    } 
    adc_test_channel =  &channels[0];    
    if (device_create_file(&pdev->dev, &measure_attr)) {
        pr_err("device create file failed\n");
        goto err;
    }
    err:
        return -1;
} 

static int rk_adc_test_remove(struct platform_device *pdev) {
    device_remove_file(&pdev->dev, &measure_attr);
    iio_channel_release(adc_test_channel);
    adc_test_channel = NULL;return 0;
} 

static const struct of_device_id rk_adc_test_match[] = {
    { .compatible = "rk-adc-test" },
    {},
}; 

MODULE_DEVICE_TABLE(of, rk_adc_test_match); 
static struct platform_driver rk_adc_test_driver = {
    .probe      = rk_adc_test_probe,
    .remove     = rk_adc_test_remove,
    .driver     = {
        .name   = "rk-adc-test",
        .owner  = THIS_MODULE,
        .of_match_table = rk_adc_test_match,
    }
}; 
module_platform_driver(rk_adc_test_driver);
```

将以上源码保存为 drivers/iio/adc/rockchip-adc-test.c ，并在 drivers/iio/adc/Makefile 后加入：

```
obj-$(CONFIG_ROCKCHIP_ADC) += rk_adc_test.o
```

编译并烧写内核和 resource.img，启动后即可在终端下运行以下命令来读取 ADC0 的值：

```
while true; do cat /sys/devices/2006c000.adc/adc_test.*/measure; sleep 1; done
```

注意，该例程并没有采用 iio_channel_get 来获取通道，而是调用 iio_channel_get_all, 读取 io-channels 属性所声明的通道列表，后取首个通道：

```
struct iio_channel *channels;channels = iio_channel_get_all(&pdev->dev);
if (IS_ERR(channels)) {
    pr_err("get adc channels fails\n");
    goto err;
} 
adc_test_channel =  &channels[0];
```

## ADC 常用函数接口

```
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

* 功能：获取 iio 通道描述
- 参数：  
    + dev: 使用该通道的设备描述指针
    + consumer_channel: 通道名称

```
void iio_channel_release(struct iio_channel *chan);
```

* 功能：释放 iio_channel_get 函数获取到的通道
- 参数：  
	+ chan：要被释放的通道描述指针
    
```
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

* 功能：读取 chan 通道 AD 采集的原始数据。
- 参数：  	
	+ chan：要读取的采集通道指针
	+ val：存放读取结果的指针