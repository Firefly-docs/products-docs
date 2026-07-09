# PWM 使用
AIO-1688JD4 开发板上对外只引出一个 PWM（FAN）, 但 Core-1688JD4 的金手指引出了多路 PWM， 在控制台下运行 PMW 读写命令或者自行在内核态或者用户态编写 PWM 读写程序，
就可以对 PWM 进行输入输出操作;

*  PWM 操作在定频时钟 100MHz，共有 20 路，每路可单独控制;
* BM1688共有 5 个 PWM IP (pwmchip0/ pwmchip4/ pwmchip8/ pwmchip12/ pwmchip16), 各 IP 控制 4 路讯号, 总共可控制 20 路讯号；

在 Linux sysfs 中, pwm0~pwm3 的 device node 各自如下:
```
/sys/class/pwm/pwmchip0/pwm0~3
```
在 Linux sysfs 中, pwm4~pwm7 的 device node 各自如下:
```
/sys/class/pwm/pwmchip4/pwm0~3
```
以风扇的设备树配置为例：
```
&fan0 {
		compatible = "pwm-fan";
		#cooling-cells = <2>;
		pwms = <&pwm2 3 1000000 0>; 
		cooling-levels = <255 170 100 1>; // max 1
};
```
其中的 `pwms = <&pwm2 3 1000000 0>; ` 表示使用 PIN 脚中的 `PWM11`, 因为 `pwm2` 为第三组，`3` 表示第四个通道，由此得出：`3 x 4 = 12(0~11)`

以此类推
## 操作示例
### PWM 操作命令示例：

1.在控制面板使用 echo 命令，配置待操作的 PWM 编号, 此例为 PWM1：
```
echo 1 > /sys/class/pwm/pwmchip0/export
```
2.设置 PWM 一个周期的持续时间，单位为 ns：
```
echo 1000000 >/sys/class/pwm/pwmchip0/pwm1/period
```
3.设置一个周期中的 "ON" 时间，单位为 ns，即占空比 `=duty_cycle/period=50%` ：
```
echo 500000 >/sys/class/pwm/pwmchip0/pwm1/duty_cycle
```
4.设置 PWM 使能
```
echo 1 >/sys/class/pwm/pwmchip0/pwm1/enable
```
### 通过文件 IO 操作程序示例：

1.配置待操作的 PWM 编号, 以 PWM1 为例：  

```
fd = open("/sys/class/pwm/pwmchip0/export", O_WRONLY);
if(fd < 0)
{
dbmsg("open export error\n");
return -1;
}
ret = write(fd, "1", strlen("0"));
if(ret < 0)
{
dbmsg("Export pwm1 error\n");
return -1;
}
```

2.设置 PWM 一个周期的持续时间，单位为 ns：  

```
fd_period = open("/sys/class/pwm/pwmchip0/pwm1/period", O_RDWR);
ret = write(fd_period, "1000000”,strlen("1000000”));
if(ret < 0)
{
dbmsg("Set period error\n");
return -1;
}
```  

3.设置一个周期中的”ON”时间，单位为 ns: (此例占空比为 50%)

```
fd_duty = open("/sys/class/pwm/pwmchip0/pwm1/duty_cycle", O_RDWR);
ret = write(fd_duty, "500000", strlen("500000"));
if(ret < 0)
{
dbmsg("Set period error\n");
return -1;
}
```  

4.设置 PWM 使能  

```
fd_enable = open("/sys/class/pwm/pwmchip0/pwm1/enable", O_RDWR);
ret = write(fd_enable, "1", strlen("1"));
if(ret < 0)
{
dbmsg("enable pwm0 error\n");
return -1;
}
```
