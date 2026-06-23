# ADC 使用
AIBOX-K3 开发板上并没有引出外部的 ADC, 但 K3-COM260 的金手指引出了多路 ADC， 在控制台下运行 ADC 读写命令或者自行在内核态或者用户态编写 ADC 读写程序，就
可以对 ADC 进行输入输出操作。

用户层通过访问 IIO 接口来实现 5 通道，12-bit ADC 的触发、采样等操作。

* 参考电压为 1.5v ref 
* adc 引脚和 sysfs 文件对应关系如下：
    * adc1 对应 sysfs 文件为 in_voltage1_raw
    * adc2 对应 sysfs 文件为 in_voltage2_raw
    * adc3 对应 sysfs 文件为 in_voltage3_raw
    * sar0 对应 sysfs 文件为 in_voltage4_raw
    * sar1 对应 sysfs 文件为 in_voltage5_raw
* 电压值计算公式：vol = val * 1500 / 4096，单位：mV

## 操作示例
### ADC 操作命令示例
1. 指定 ADC 通道 1~5, 此例为 ADC1:
```
echo 1 > /sys/bus/iio/devices/iio:device0/in_voltage1_raw
```

2. 读出刚才指定的 ADC channel 值:
```
cat /sys/bus/iio/devices/iio:device0/in_voltage1_raw
```

### 用户态 ADC 读取操作程序示例
参考如下：
```
fd = open("/sys/bus/iio/devices/iio:device0/in_voltage1_raw", O_RDWR|
O_NOCTTY|O_NDELAY);
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
