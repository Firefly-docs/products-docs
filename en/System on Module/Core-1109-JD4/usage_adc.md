# ADC

## Introduction

Core-1109-JD4 There are two AD interfaces on the development board: temperature sensor (Temperature Sensor) and successive approximation ADC (Successive Approximation Register). Among them:

*    TS-ADC (Temperature Sensor): supports 2 channels.
*    SAR-ADC (Successive Approximation Register): supports six-channel single-ended 10-bit SAR-ADC, with a maximum conversion rate of 1MSPS and a 20MHz A/D converter clock.

The core uses an industrial I/O subsystem to control the ADC, which is mainly designed for sensors with AD conversion or DA conversion. The following takes SAR-ADC as an example to introduce the basic configuration method of ADC.

## DTS configuration

### Configure DTS nodes
The SAR-ADC nodes of Core-1109-JD4 defined in `kernel/arch/arm/boot/dts/rv1126.dtsi` file, as showm below:

```
saradc: saradc@ff5e0000 {
	compatible = "rockchip,rk3399-saradc";
	reg = <0xff5e0000 0x100>;
	interrupts = <GIC_SPI 40 IRQ_TYPE_LEVEL_HIGH>;
	#io-channel-cells = <1>;
	clocks = <&cru CLK_SARADC>, <&cru PCLK_SARADC>;
	clock-names = "saradc", "apb_pclk";
	resets = <&cru SRST_SARADC_P>;
	reset-names = "saradc-apb";
	status = "disabled";
};
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

For example, if the standard voltage is 1.8V, the AD acquisition bit number is 12 bits, and the raw data collected by AD is 445, then:

```
Vresult = (1800mv * 445) / 4095;
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

### Gets all ADC values

There is a convenient way to query the value of each SARADC:

```
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```

## FAQs

### Follow the steps above to apply for SARADC. Why is there an application error?

When the driver needs to get the ADC channel to use, it needs to control the load time of the driver, which must be after saradc initialization. Saradc uses `module_platform_driver()` for platform device driver registration and ultimately calls `module_init()`. Therefore, the driver loading function of the user only needs to use one with lower priority than `module_init()`, such as `late_initcall()`, so as to ensure that the loading time of the driver is later than the initialization time of saradc and avoid errors.
