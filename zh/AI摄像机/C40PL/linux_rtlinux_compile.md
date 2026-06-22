# 升级Linux为RTLinux

为了满足实时需求，我们在SDK源码的内核基础上升级Linux到RTLinux。

## 下载RTlinux固件
我们提供了rtlinux固件下载：[rtlinux Ubuntu固件(64位)]()

（有反馈说在使用hdmi测试实时性能，最达延时有超过200us的现象。
为此我们重新优化了内核，修复了使用hdmi出现高延迟现象，并提升实时性能。）

## 测试实时效果
测试实时性能需要cyclictest,可以使用apt安装。

```
apt update
apt install rt-tests

```
使用apt安装的版本比较低（v1.0），可以下载源码编译，建议在板子里编译，因为使用交叉编译工具编译出来的cyclictest功能不完整，比如-a选项无法绑定某些CPU，只能绑定某个或者全部CPU
```
#下载rt_tests源码
git clone git://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git
#安装依赖
sudo apt-get install build-essential libnuma-dev
#切换分支：
git checkout v1.7
```
在板子上编译直接make；在其他平台编译时需要指定交叉编译工具路径。如下：
```
make CROSS_COMPILE=/home/dengkx/project/rk356x/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
```
然后执行，生成cyclictest。


## 测试RTLinux的实时效果
拷贝cyclictest到板子上，执行以下命令对20个线程进行实时测试。
```
#加上-a 实时效果会更好，版本比较低时需要加上-n选项
sudo ./cyclictest -t 20 -p 99
```
以下的的截图是minimal ubuntu在cpu满压力下测得的结果，T:0 序号为0的线程；P:99 线程优先级为99
；C: 9397 计数器。线程的时间间隔每达到一次，计数器加1；I: 1000 时间间隔为1000微秒(us)
；Min: 最小延时(us)；Act: 最近一次的延时(us)；Avg：平均延时(us)；Max： 最大延时(us)。

![](../../../rv1126_img/RK3588_RTLinux_text.png)

## 一些使用参数解析:
```
-t:指定运行多少个实时线程 ；	例如运行20个实时线程: -t20
-p:指定运行的实时线程优先级；	例如运行线程的优先级为80: -p80
-a:设置实时线程亲和核心；		例如设置实时线程运行在1到3CPU上: -a 1-3（后面不加参数，则指定全部核心；低版本指定单个核心或者全部核心）
-i:线程的基础间隔，默认为1000us
-l;线程之间的间隔，默认为500us
-m:固定程序到内存,防止程序被调出内存
-q:只在程序结束时打印结果
```

## Cyclictest标准测试

threads选项(-t)用于指定Cyclictest在检测延迟时将使用的测量线程数。通常，在系统上的每个CPU上只运行一个测量线程是一个标准的测试方案。可以使用亲和性选项(-a)指定线程必须在其上执行的cpu。

这些选项对于最小化运行Cyclictest对观察到的系统的影响至关重要。在使用Cyclictest时，确保在任何给定时间只执行一个测量线程是很重要的。如果两个或多个Cyclictest线程的预期执行时间重叠，则Cyclictest的测量将受到其自己的测量线程所造成的延迟的影响。确保在给定的时间只执行一个测量线程的最好方法是在给定的CPU上只执行一个测量线程。

例如，如果要分析三个特定cpu的延迟，则指定应该使用这些cpu(使用-a选项)，并指定应该使用三个测量线程(使用-t选项)。在这种情况下，为了最小化Cyclictest的开销，请确保收集度量数据的主Cyclictest线程没有运行在三个隔离的cpu之一上。主线程的关联性可以使用taskset程序设置，如下所述。

### 在评估一组隔离的cpu上的延迟时，减小cyclictest的影响

在测量cpu子集上的延迟时，确保主Cyclictest线程正在未被评估的cpu上运行。例如，如果一个系统有两个CPU，并且正在评估CPU 0上的延迟，那么主Cyclictest线程应该固定在CPU 1上。Cyclictest的主线程不是实时的，但是如果它在被评估的CPU上执行，它可能会对延迟产生影响，因为会有额外的上下文切换。在启动Cyclictest之后，可以使用taskset命令将主线程限制为在cpu的某个子集上执行。例如，针对CPU1到CPU3的延时测试:
```
#CPU1到CPU3运行实时程序，主线运行在CPU0上（需要板子编译出来的cyclictest，否则三个实时线程只会运行在CPU1上，CPU2和3则没有）
taskset -c 0 ./cyclictest -t3 -p99 -a 1-3
```
taskset程序还可以用于确保系统上运行的其他程序不会影响隔离CPU上的延迟。例如，启动程序top查看线程并固定到CPU 0上，使用下面的命令:

```
taskset --cpu 0 top -H -p PID
#top打开之后点击f键，光标移动到 P 选项，空格选中，然后点击 q建退出，便可查看到实时线程在哪些CPU上运行。
```

## 在最坏的情境下测试CPU延迟

下面一条的测试代码是RT系统中最坏情况下的背景压力。
```
time ./cyclictest -t50 -p 80 -i 10000 -n -l 100000000000 -d 86400  -a 3
```

你还可以制造相应的压力环境来进行测试CPU延时，例如：
使用stress制造四个核心100%CPU负载。
```
#stress -c 4& 
#还可以固定到CPU上，如下：
taskset -c 0 stress -c 1&
taskset -c 1 stress -c 1&
taskset -c 2 stress -c 1&
taskset -c 3 stress -c 1&
```

也可以使用ping制造100%CPU负载。
```
taskset -c 0 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 1 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 2 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 3 /bin/ping -l 100000 -q -s 10 -f localhost &
```

对于I/O负载可以使用tar来制造。例如下载linux源码git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux-stable
```
cd /opt
while true; do taskset -c 0 tar cvzf test1.tgz  ./linux-stable ; done  &
while true; do taskset -c 1 tar cvzf test2.tgz  ./linux-stable ; done  &
while true; do taskset -c 2 tar cvzf test3.tgz  ./linux-stable ; done  &
while true; do taskset -c 3 tar cvzf test4.tgz  ./linux-stable ; done  &
```
产生网络负载。
```
# 启动服务器
/usr/bin/netserver
# 启动 netperf 连接
/usr/bin/netperf -H <IP_ADDR_OF_SERVER> -t TCP_STREAM -A 16K,16K -l 3600
# 使用浏览器
firefox http://www.intomail.net/stream.php
```

根据需求来选择压力环境，然后进行延时测试。


## 一些使用rtlinux的建议
### 抑制控制台消息及禁止内存过度使用
```
#可以使用内核参数quiet启动内核，或者启动之后抑制，如下：
echo 1 > /proc/sys/kernel/printk

#禁用内存过度使用以避免 Out-of-Memory Killer产生的延迟
echo 2 > /proc/sys/vm/overcommit_memory
``` 

### 不使用桌面或者使用轻量级桌面

为了更好的实时，我们不建议使用带桌面的系统，因为这将为CPU延迟带来很大的挑战。建议使用minimal ubuntu、自己的QT程序等。

### 特定的核心处理特定的事

实时要求高的事件固定到某个核心上处理，系统及其它实时要求不高的事件集中到一个核心上处理。例如特定的中断，实时程序等事件可以用专门的核心为他们服务。

### 使用芯片的MCU

对于实时要求更高的事件，建议使用芯片上的MCU功能，以便达到更好的实时控制。

## CANFD驱动
目前暂时只有rtlinux使用CANFD驱动,CAN FD是兼容CAN功能的，发送CAN格式就可以。由于rk3588芯片的canfd发送数据格式有问题，所以只使用can功能。
如果你一定要使用canfd功能，你需要保证发送报文的最后五位数据不全为0或1（例如，将最后两位数据固定为01等），便可以正常使用canfd功能。

```
#在收发端关闭can0设备
ip link set can0 down

#在收发端设置仲裁段1M波特率,数据段3M波特率                                          
ip link set can0 type can bitrate 1000000 dbitrate 3000000 fd on

#打印can0信息
ip -details link show can0

#在收发端打开can0设备
ip link set can0 up

#在接收端执行candump,阻塞等待报文                          	                 
candump can0

#canfd格式,在发送端执行cansend,发送报文
cansend can0 123##112345678

#can格式,在发送端执行cansend,发送报文                                     
cansend can0 123#1122334455667788                          
```
