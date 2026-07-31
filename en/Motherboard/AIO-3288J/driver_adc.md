# ADC Use

## Introduction

The AD interface on the AIO-3288J development board is divided into high-speed ADC stream interface, temperature sensor and successive approximation register. This paper mainly introduces the basic configuration methods of ADC.

An industrial I/O subsystem is used by the kernel to control the ADC, and the subsystem is primarily designed for the sensor of AD conversion or DA conversion.

Its associated data structures and configuration methods are as follows:

## Data structure

### iio_channel structure

```c
struct iio_channel {
    struct iio_dev *indio_dev;//industrial I/O device
    const struct iio_chan_spec *channel;//I/O channel
    void *data;
};
```

### iio_dev structure

This structure is mainly used to describe the device to which the IO port belongs, and its specific definition is as follows:

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

### iio_chan_spec structure

This structure is mainly used to describe the properties of a single channel, and its specific definition is as follows:

```c
struct iio_chan_spec {
    enum iio_chan_type  type; // describe the channel type
    int         channel; //channel number
    int         channel2; //channel number
    unsigned long       address; //channel address
    int         scan_index;
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

## Configuration steps

### Configure the DTS node

The DTS node for the Firefly ADC is defined in the `kernel/arch/arm/boot/dts/rk3288.dtsi` file, it is shown as follows:

```dts
adc: adc@ff100000 {
    compatible = "rockchip,saradc";
    reg = ;
    interrupts = ;
    #io-channel-cells = ;
    io-channel-ranges;rockchip,adc-vref = ;
    clock-frequency = ;
    clocks = , ;
    clock-names = "saradc", "pclk_saradc";
    status = "disabled";
};
```

The user only needs to add the channel definition to the firefly-rk3288-aio-3288j.dts file and change its status to "okay":

```dts
&adc {
    status = "okay";
    adc_test{
        compatible = "rockchip,adc_test";
        io-channels = ;
    };
};
```

### Match the DTS node in the driver file

Match the DTS node in the driver file:

```c
static const struct of_device_id of_XXX_match[] = {
    { .compatible = "rockchip,adc_test" },
    { /* Sentinel */ }
};
```

Populate the structure array into the platform_driver that the ADC is used.

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

### Get the AD Channel

```c
struct iio_channel *chan; //define the IIO channel structure
chan = iio_channel_get(&pdev->dev, NULL); //get the IIO channel structure
```

Note: iio_channel_get gets the IIO channel structure through the parameter pdev passed in by the probe function, and the probe function is as follows:

```c
static int XXX_probe(struct platform_device *pdev);
```

### Read the raw data acquisited by AD

```c
int val,ret;
ret = iio_read_channel_raw(chan, &val);
```

The iio_read_channel_raw function is called to read the raw data acquisited by AD and store it in val.

### Calculate the voltage acquisited

The standard voltage is used to convert the value converted by AD to the desired voltage value for the user. It is calculated as follows:

```c
Vref / (2^n-1) = Vresult / raw
```

* Note:
    + Vref is the standard voltage
    + n is the number of bits converted by AD
    + Vresult is the acquisition voltage required by the user
    + raw is the raw data acquisited by the AD

For example, the standard voltage is 1.8V, the number of bits of AD acquisition is 10, and the raw data acquisited by AD is 568, then:

```c
Vresult = (1800mv * 568) / 1023;
```

## ADC common function interface

```c
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

* Function: get the iio channel description
* Parameter：
    + dev: pointer to the device description using the channel
    + consumer_channel: pointer to IIO channel description used by the device

```c
void iio_channel_release(struct iio_channel *chan);
```

* Function: release the the channel acquired by the iio_channel_get function
* Parameters:
    + chan：pointer to the channel description to be released

```c
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

* Function: read the raw data acquisited by chan channel AD.
* Parameters:
    + chan：pointer to the acquisition channel to be read
    + val：pointer to store the reading results