# ADC Usage

## Introduction [step]

The AD interface on the board is a Successive Approximation Register ADC (SAR-ADC), single-ended 12-bit. The kernel controls the ADC through the Industrial I/O subsystem (IIO), which is designed for AD/DA conversion sensors. The following uses the SAR-ADC as an example to describe the basic usage of the ADC of Firefly-R3572A.

## Hardware Connection [step]

The ADC signal is available on the J1 header with the following pins:

| Pin NO. | Signal | Level |
| --- | --- | --- |
| 15 | SARADC_VIN4 | 1.8 V |
| 5 / 6 | GND |  |

<center>

![](../../../../rk3572_img/Firefly-R3572A/interface_extension.jpg)
</center>

<Callout title="Note" type="warn">

The SARADC input level is 1.8 V. Do not exceed this range, otherwise the chip may be damaged.

</Callout>

## DTS Configuration [step]

### Configure the DTS Node

The SARADC controller node is defined in the chip-level DTS of the SDK. The board-level DTS requests the channel to be used. Example for channel 4 (SARADC_VIN4), adding the ADC resource description in the DTS file:

```text
adc_demo: adc_demo {
    compatible = "firefly,rk3572-adc";
    status = "okay";
    io-channels = <&saradc 4>;
};
```

Refer to the actual Firefly-R3572A SDK code for the DTS file path and the compatible string.

### Match the DTS Node in the Driver

For the user driver, refer to the Firefly adc demo. First define the `of_device_id` structure array in the driver file:

```c
static const struct of_device_id firefly_adc_match[] = {
    { .compatible = "firefly,rk3572-adc" },
    {},
};
```

Then fill this structure array into the platform_driver that uses the ADC:

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

Then parse the resources added in the DTS in firefly_adc_probe:

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

## Driver Description [step]

### Get the AD Channel

```c
struct iio_channel *chan;     // define the IIO channel structure
chan = iio_channel_get(&pdev->dev, NULL);    // get the IIO channel structure
```

**Note:** `iio_channel_get` gets the IIO channel structure through the pdev parameter passed in by the probe function:

```c
static int XXX_probe(struct platform_device *pdev);
```

### Read the Raw AD Data

```c
int val, ret;
ret = iio_read_channel_raw(chan, &val);
```

Call iio_read_channel_raw to read the raw AD data into val.

### Calculate the Collected Voltage

Use the standard voltage to convert the AD conversion value into the required voltage. The formula is:

```text
Vref / (2^n-1) = Vresult / raw
```

Notes:

* Vref is the standard voltage
* n is the number of AD conversion bits
* Vresult is the required collected voltage
* raw is the raw data collected by the AD

For example, with a standard voltage of 1.8V, 12-bit AD conversion, and a raw value of 445:

```text
Vresult = (1800mv * 445) / 4095;
```

## Interface Description [step]

```c
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

* Function: get the iio channel description
* Parameters:
  * dev: pointer to the device description using the channel
  * consumer_channel: pointer to the IIO channel description used by the device

```c
void iio_channel_release(struct iio_channel *chan);
```

* Function: release the channel obtained by iio_channel_get
* Parameters:
  * chan: pointer to the channel description to be released

```c
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

* Function: read the raw AD data of the chan channel.
* Parameters:
  * chan: pointer to the channel to read
  * val: pointer to store the result

## Debug Methods [step]

### Read the ADC Value

After the system boots, read the raw ADC data through the IIO device node:

```bash
cat /sys/bus/iio/devices/iio\:device0/in_voltage4_raw
```

The device node number depends on the actual system. Confirm it with `ls /sys/bus/iio/devices/`.

### Get All ADC Values

There is a convenient way to query the value of each SARADC:

```bash
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```

## FAQs [step]

### Follow the steps above to apply for the SARADC. Why is there an application error?

When the driver needs to get the ADC channel to use, it needs to control the load time of the driver, which must be after the saradc initialization. The saradc uses `module_platform_driver()` for platform device driver registration, which finally calls `module_init()`. So the user driver only needs to use a load priority lower than `module_init()`, such as `late_initcall()`, to ensure that the driver is loaded later than the saradc initialization and avoid errors.
