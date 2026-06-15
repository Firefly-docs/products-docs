# ADC

## Introduction

The AIO-3568J development board has two kinds of AD interface, respectively: Temperature Sensor and Successive Approximation Register. Among them:

* TS-ADC (Temperature Sensor): Supports two-channel, clock frequency must be less than 800KHZ.
* SAR-ADC (Successive Approximation Register): Supports six-channel single-ended 10-bit SAR-ADC. Must have a clock frequency less than 13MHZ.

The kernel uses the industrial I/O subsystem to control the ADC, which is mainly designed for AD conversion or DA conversion sensor. The following is an example of SAR-ADC using ADC fan to introduce the basic configuration method of ADC.

## DTS configuration

### Configure DTS nodes
The SAR-ADC nodes of AIO-3568J defined in `kernel/arch/arm64/boot/dts/rockchip/rk3568.dtsi` file, as showm below:

```
saradc: saradc@fe720000 {
    compatible = "rockchip,rk3568-saradc", "rockchip,rk3399-saradc";
    reg = <0x0 0xfe720000 0x0 0x100>;
    interrupts = <GIC_SPI 93 IRQ_TYPE_LEVEL_HIGH>;
    #io-channel-cells = <1>;
    clocks = <&cru CLK_SARADC>, <&cru PCLK_SARADC>;
    clock-names = "saradc", "apb_pclk";
    resets = <&cru SRST_P_SARADC>;
    reset-names = "saradc-apb";
    status = "disabled";
};
```

Firstly, user needs to add the resource description of ADC in the DTS file:

Linux DTS: kernel/arch/arm64/boot/dts/rockchip/.dts

Android DTS: kernel/arch/arm64/boot/dts/rockchip/.dts
```
   adc_demo: adc_demo{
       status = "disabled";
       compatible = "firefly,rk356x-adc";
       io-channels = <&saradc x>;	// x is channel index, 0 means using SARADC channel 0
   };
```


### Matches DTS nodes in the driver file

The user driver can refer to Firefly adc demo: `kernel/drivers/iio/adc/adc-firefly-demo.c`, which is a driver that detects the status of AIO-3568J's fan. First, define the `of_device_id` structure array in the driver file:

```
static const struct of_device_id firefly_adc_match[] = {
     { .compatible = "firefly,rk356x-adc" },
     {},
};
```

Then populate the structure array with `platform_driver` to use ADC:

```
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

Then, in `firefly_adc_probe`, the added resource of DTS is parsed:

```
static int firefly_adc_probe(struct platform_device *pdev)
{
    printk("firefly_adc_probe!\n");
    chan = iio_channel_get(&(pdev->dev), NULL);
    if (IS_ERR(chan)){
        chan = NULL;
        printk("%s() have not set adc chan\n", __FUNCTION__);
        return -1;
    }
    fan_insert = false;
    if (chan) {
        INIT_DELAYED_WORK(&adc_poll_work, firefly_demo_adc_poll);
        schedule_delayed_work(&adc_poll_work,1000);
    }
    return 0;
}
```

## Drive instructions

### Get AD channel

```
struct iio_channel *chan;                    #Defines the IIO channel structure
chan = iio_channel_get(&pdev->dev, NULL);    #Get AD channel structure
```

**Note:** `iio_channel_get` gets the IIO channel structure through the parameter pdev passed in by the probe function. The probe function is as follows:

```
static int XXX_probe(struct platform_device *pdev);
```

### Read the original data collected by AD

```
int val,ret;
ret = iio_read_channel_raw(chan, &val);
```

Call the `iio_read_channel_raw` function to read the raw data collected by AD and store it in `val`.

### Calculate the collected voltage

The standard voltage is used to convert the value of AD to the voltage value required by the user. The calculation formula is as follows:

```
Vref / (2^n-1) = Vresult / raw
```

Note:

* Vref is the standard voltage
* n is the number of bits converted to AD
* Vresult is the collection voltage required by the user
* raw is the original data that AD collects

For example, the standard voltage is 1.8V, the collection bits of AD is 10, and the original data collected by AD is 568, then:

```
Vresult = (1800mv * 568) / 1023;
```

## Interface specification

```
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

- **Function** : Gets the iio channel description.
- **Parameters** :
     + **dev**: Using the device description pointer of this channel.
     + **consumer_channel**: The IIO channel description pointer used by the device.

```
void iio_channel_release(struct iio_channel *chan);
```

- **Function** : Release the channel obtained by the `iio_channel_get` function.
- **Parameters** :
     + **chan** ：The channel description pointer to be released.

```
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

- **Function** : Read the original data collected by chan channel AD.
- **Parameters** :
     + **chan**：Collection channel pointer to read.
     + **val**：The pointer to the result of reading.

## Debug method

### Demo program

Enable adc_demo in the DTS, change "disabled" to "okay" :

```
adc_demo: adc_demo{
        status = "okay";
        compatible = "firefly,rk356x-adc";
        io-channels = <&saradc 5>;
};
```

Compile the kernel, upgrade the kernel to AIO-3568J development board, and then plug out the fan, the kernel log information will be printed as follows:

```
[   85.158104] Fan insert! raw= 135 Voltage= 237mV
[   88.422124] Fan out! raw= 709 Voltage=1247mV
```

### Gets all ADC values

There is a convenient way to query the value of each SARADC:

```
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```

## FAQs

### Follow the steps above to apply for SARADC. Why is there an application error?

When the driver needs to get the ADC channel to use, it needs to control the load time of the driver, which must be after saradc initialization. Saradc uses `module_platform_driver()` for platform device driver registration and ultimately calls `module_init()`. Therefore, the driver loading function of the user only needs to use one with lower priority than `module_init()`, such as `late_initcall()`, so as to ensure that the loading time of the driver is later than the initialization time of saradc and avoid errors.
