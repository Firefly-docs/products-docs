# PWM Usage
The AIO-186JD4 development board only exposes one PWM (FAN) externally, but the AIBOX-186's gold fingers expose multiple PWMs. By running PMW read/write commands in the console or writing PWM read/write programs in kernel or user mode, you can perform input and output operations on the PWM.

* PWM operates at a fixed frequency clock of 100MHz, with a total of 20 channels, each of which can be controlled individually.
* The CV186AH has 5 PWM IPs (pwmchip0/ pwmchip4/ pwmchip8/ pwmchip12/ pwmchip16), each controlling 4 channels, allowing for a total of 20 channels.

In the Linux sysfs, the device nodes for pwm0~pwm3 are as follows:
```
/sys/class/pwm/pwmchip0/pwm0~3
```
In the Linux sysfs, the device nodes for pwm4~pwm7 are as follows:
```
/sys/class/pwm/pwmchip4/pwm0~3
```
Taking the fan's device tree configuration as an example:
```
&fan0 {
		compatible = "pwm-fan";
		#cooling-cells = <2>;
		pwms = <&pwm2 3 1000000 0>; 
		cooling-levels = <255 170 100 1>; // max 1
};
```
Here, `pwms = <&pwm2 3 1000000 0>;` indicates the use of the `PWM11` pin, as `pwm2` is the third group, and `3` indicates the fourth channel, leading to the conclusion: `3 x 4 = 12(0~11)`

And so on.
## Operation Example
### Example of PWM Operation Commands:

* Use the echo command on the control panel to configure the PWM number to be operated on, in this case PWM1:
```
echo 1 > /sys/class/pwm/pwmchip0/export
```
* Set the duration of one PWM cycle, in nanoseconds:
```
echo 1000000 >/sys/class/pwm/pwmchip0/pwm1/period
```
* Set the "ON" time within one cycle, in nanoseconds, i.e., duty cycle `=duty_cycle/period=50%`:
```
echo 500000 >/sys/class/pwm/pwmchip0/pwm1/duty_cycle
```
* Enable the PWM:
```
echo 1 >/sys/class/pwm/pwmchip0/pwm1/enable
```
### Example of File IO Operation Program:

* Configure the PWM number to be operated on, taking PWM1 as an example:
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

* Set the duration of one PWM cycle, in ns:

```
fd_period = open("/sys/class/pwm/pwmchip0/pwm1/period", O_RDWR);
ret = write(fd_period, "1000000", strlen("1000000"));
if(ret < 0)
{
dbmsg("Set period error\n");
return -1;
}
```

* Set the "ON" time within one cycle, in ns: (In this example, the duty cycle is 50%)

```
fd_duty = open("/sys/class/pwm/pwmchip0/pwm1/duty_cycle", O_RDWR);
ret = write(fd_duty, "500000", strlen("500000"));
if(ret < 0)
{
dbmsg("Set period error\n");
return -1;
}
```

* Enable PWM

```
fd_enable = open("/sys/class/pwm/pwmchip0/pwm1/enable", O_RDWR);
ret = write(fd_enable, "1", strlen("1"));
if(ret < 0)
{
dbmsg("enable pwm0 error\n");
return -1;
}
```