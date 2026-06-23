# ADC Usage
The AIO-186JD4 development board does not expose an external ADC, but the AIBOX-186's gold fingers expose multiple ADCs. By running ADC read and write commands in the console or writing ADC read and write programs in kernel mode or user mode, input and output operations on the ADC can be performed.

The user layer accesses the IIO interface to implement triggering, sampling, and other operations for a 5-channel, 12-bit ADC.

* Reference voltage is 1.5v ref
* The correspondence between ADC pins and sysfs files is as follows:
    * adc1 corresponds to sysfs file in_voltage1_raw
    * adc2 corresponds to sysfs file in_voltage2_raw
    * adc3 corresponds to sysfs file in_voltage3_raw
    * sar0 corresponds to sysfs file in_voltage4_raw
    * sar1 corresponds to sysfs file in_voltage5_raw
* Voltage value calculation formula: vol = val * 1500 / 4096, unit: mV

## Operation Example
### ADC Operation Command Example
1. Specify ADC channel 1~5, this example is for ADC1:
```
echo 1 > /sys/bus/iio/devices/iio:device0/in_voltage1_raw
```

2. Read the value of the specified ADC channel:
```
cat /sys/bus/iio/devices/iio:device0/in_voltage1_raw
```

### User Mode ADC Read Operation Program Example
Refer to the following:
```
fd = open("/sys/bus/iio/devices/iio:device0/in_voltage1_raw", O_RDWR|O_NOCTTY|O_NDELAY);
if (fd < 0)
printf("open adc err!\n");
write(fd, "1", 1);
lseek(fd, -1, SEEK_CUR);
char buffer[512] = {0};
int len = 0;
unsigned int adc_value = 0;
len = read(fd, buffer, 5);
if (len != 0) {
printf("read buf: %s\n", buffer);
adc_value= atoi(buffer);
printf("adc value is %d\n", adc_value);
}
write(fd, "0", 1);
close(fd);
```