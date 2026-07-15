# ADC Use

## Introduction
There's a SAR ADC (Successive Approximation Register Analog Digital Converter) in Firefly-RK3128 development, with three 10 bits channels (0/1/2):

* ADCIN0: exported in the extension board
* ADCIN1: use as Recovery key detection internally
* ADCIN2: exported in the extension board

This article will introduce how to configure the ADC to work properly.

## Configure the DTS Node

The DTS device node of the ADC is already defined in file kernel/arch/arm/boot/dts/rk312x.dtsi:
```
adc: adc@2006c000 {
	compatible = "rockchip,saradc";
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

You need to add a new node to rk3128-fireprime.dts,  with status set to "okay":
```
&adc {
    status = "okay";
    adc_test {status = "okay";
    compatible = "rk-adc-test";
    io-channels = <&adc 0>;
};
```

New test node adc_test is added above, for later use of the test code below.
Source code of ADC driver is drivers/iio/adc/rockchip_adc.c

## Get the ADC channel
```
struct iio_channel *chan;chan = iio_channel_get(&pdev->dev, "adc0");
```

adc0 is the channel name, which is defined in rockchip_adc.c:
```
static const struct iio_chan_spec rk_adc_iio_channels[] = {ADC_CHANNEL(0, "adc0"),ADC_CHANNEL(1, "adc1"),ADC_CHANNEL(2, "adc2"),ADC_CHANNEL(6, "adc6"),};
```

### Read the raw data acquisited by AD
```
int val, ret;ret = iio_read_channel_raw(chan, &val);
```

Call iio_read_channel_raw to read the raw data from the channel and store the result in val parameter.

### Calculate the voltage acquisited

Use the standard voltage to convert the raw data to the result voltage. The equation is shown as below:
```
Vref / (2^n-1) = Vresult / raw
```

Note:  

* Vref is the standard voltage
* n is the AD conversion accuracy, such as 10 bits or 12 bits.
* Vresult is the conversion result of the voltage.
* raw is the raw data read from function iio_read_channel_raw.

For example, the standard voltage is 1.8V, n is 10 bits, raw data is 568. The conversion expression is:
```
Vresult = (1800mv * 568) / 1023;
```

## Full Test Code

Here is the full list of ADC test code:
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

Save the source code above as  drivers/iio/adc/rockchip-adc-test.c , and append the following line to drivers/iio/adc/Makefile :
```
obj-$(CONFIG_ROCKCHIP_ADC) += rk_adc_test.o
```

Compile and flash the kernel.img and resource.img. When the board boots, enter the following command to read the ADC0 channel value:
```
while true; do cat /sys/devices/2006c000.adc/adc_test.*/measure; sleep 1; done
```

Note that, this demo uses iio_channel_get_all instead of use iio_channel_get to get ADC channel. It reads the channel list declared by io-channels property, and take the first one:
```
struct iio_channel *channels;channels = iio_channel_get_all(&pdev->dev);
if (IS_ERR(channels)) {
    pr_err("get adc channels fails\n");
    goto err;
} 
adc_test_channel =  &channels[0];
```

## ADC API
```
/**
 * iio_channel_get() - get description of all that is needed to access channel.
 * @dev:    Pointer to consumer device. Device name must match
 *          the name of the device as provided in the iio_map
 *          with which the desired provider to consumer mapping
 *          was registered.
 * @consumer_channel:   Unique name to identify the channel on the consumer
 *          side. This typically describes the channels use within
 *          the consumer. E.g. 'battery_voltage'
 */
 struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
 
 
 /**
 * iio_channel_release() - release channels obtained via iio_channel_get
 * @chan:       The channel to be released.
 */
 void iio_channel_release(struct iio_channel *chan);
 
 
 /**
 * iio_read_channel_raw() - read from a given channel
 * @chan:       The channel being queried.
 * @val:        Value read back.
 *
 * Note raw reads from iio channels are in adc counts and hence
 * scale will need to be applied if standard units required.
 */
 int iio_read_channel_raw(struct iio_channel *chan, int *val);
```
