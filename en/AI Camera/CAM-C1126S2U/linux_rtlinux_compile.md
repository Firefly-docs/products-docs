# Upgrade Linux to RTLinux

In order to meet real-time requirements, we upgraded Linux to RTLinux based on the kernel of SDK source code.

## Download RTlinux firmware
We provide rtlinux firmware download：[rtlinux Ubuntu firmware(64-bit)]()

( There has been feedback that the maximum latency is over 200us when testing real-time performance using HDMI.We re-optimized the kernel to fix high latency with HDMI and improve real-time performance.)

## Test real-time effects.
Testing real-time performance requires cyclictests, which can be installed using apt.
```
apt update
apt install rt-tests

```
The version of apt installation is relatively low (V1.0), so you can download the source code and compile it. It is recommended to compile it in the board, because cyclictest compiled by cross-compilation tool is not complete. For example, the -A option cannot bind some cpus, but only some or all cpus

```
#Download the rt_tests source code.
git clone git://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git
#Switch branch：
git checkout v1.7
```
Compile directly on board make;You need to specify the cross-compilation tool path when compiling on other platforms. The following. 
```
export CC=/home/dengkx/project/rk356x/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc
```
Then execute the make command to generate cyclictest.

## Test the real-time effect of RTLinux.
Copy cyclictest to the board and execute the following command to test 20 threads in real time.
```
#Add -a for better real-time effect, or -n for lower versions.
sudo ./cyclictest -t 20 -p 99
```
The following screenshot is the result measured under the full pressure of the cpu,
T:0 thread with serial number 0; P:99 thread priority is 99; C: 9397 counter.
Each time the thread interval is reached, the counter is incremented by 1;
I: 1000 time interval is 1000 microseconds (us); Min: minimum delay (us);
Act: most recent delay (us); Avg: average delay  (us); Max: Maximum delay (us).

![](../../../rv1126_img/RK3588_RTLinux_text.png)

## Some use parameter parsing:
```
-t: specifies the number of real-time threads to run. For example, running 20 real-time threads: -t20
-p: specifies the priority of the running real-time thread. For example, the priority of the running thread is 80: -p80
-a: Sets the real-time thread affinity core. For example, to set the real-time thread to run on 1 to 3 cpus: -a 1-3 (no arguments, then specify all cores; Lower versions specify single or all cores.)
-i: basic interval of threads. The default value is 1000
-l; Interval between threads, default is 500us
-m: fixes the program to the memory to prevent the program from being called out
-q: Prints the result only at the end of the program
```


## Cyclictest The standard test

The Threads option (-t) is used to specify the number of measurement threads that Cyclictest will use when detecting delays. In general, it is a standard test scenario to run only one thread of measurement on each CPU on the system. You can use the affinity option (-a) to specify the CPU on which the thread must execute.

These options are critical to minimize the impact of running cyclictests on the observed system. When using Cyclictests, it is important to ensure that only one thread of measurement is executing at any given time. If the expected execution times of two or more Cyclictest threads overlap, Cyclictest measurements are affected by delays caused by their own measurement threads. The best way to ensure that only one measurement thread executes at a given time is to execute only one measurement thread on a given CPU.

For example, if you want to analyze the latency of three specific cpus, specify that these cpus should be used (using the -A option), and specify that three measurement threads should be used (using the -t option). In this case, to minimize the Cyclictest overhead, make sure that the primary Cyclictest thread collecting the metrics is not running on one of the three isolated cpus. The correlation of the main thread can be set using the Taskset program, as described below.

###Reduce the impact of Cyclictests when evaluating latency on a set of isolated cpus

When measuring latency on a subset of cpus, make sure the primary Cyclictest thread is running on a CPU that has not been evaluated. For example, if a system has two cpus and is evaluating latency on CPU 0, the primary Cyclictest thread should be anchored to CPU 1. The main thread of a Cyclictest is not real-time, but if it is executed on the CPU being evaluated, it can have an impact on latency because of additional context switches. After Cyclictest is started, you can use the Taskset command to limit the main thread to execute on a subset of the CPU. For example, a delay test for CPU1 to CPU3:
```
#CPU1 to CPU3 run real-time programs, the main line runs on CPU0 (cyclictest compiled by the board is required, otherwise the three real-time threads will only run on CPU1, CPU2 and 3 do not)
taskset -c 0 ./cyclictest -t3 -p99 -a 1-3
```

The Taskset program can also be used to ensure that other programs running on the system do not affect latency on the isolated CPU. For example, to start the program top to view the thread and anchor it to CPU 0, use the following command:

```
taskset --cpu 0 top -H -p PID
#After opening top, click f, move the cursor top, select space, and then click q to exit. You can see which cpus the real-time thread is running on.
```

##Test CPU latency in the worst-case scenario

The following test code is the worst case background pressure on an RT system
```
time ./cyclictest -t50 -p 80 -i 10000 -n -l 100000000000 -d 86400  -a 3
```

You can also create pressure conditions to test CPU latency, such as:
Create four core 100%CPU loads using stress.
```
#stress -c 4&
#Can be fixed to the CPU, as follows:
taskset -c 0 stress -c 1&
taskset -c 1 stress -c 1&
taskset -c 2 stress -c 1&
taskset -c 3 stress -c 1&
```

You can also use ping to create 100%CPU load.
```
taskset -c 0 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 1 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 2 /bin/ping -l 100000 -q -s 10 -f localhost &
taskset -c 3 /bin/ping -l 100000 -q -s 10 -f localhost &
```

For I/O loads, tar can be used. For example, download the Linux source code:git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux-stable
```
cd /opt
while true; do taskset -c 0 tar cvzf test1.tgz  ./linux-stable ; done  &
while true; do taskset -c 1 tar cvzf test2.tgz  ./linux-stable ; done  &
while true; do taskset -c 2 tar cvzf test3.tgz  ./linux-stable ; done  &
while true; do taskset -c 3 tar cvzf test4.tgz  ./linux-stable ; done  &
```
Generate network loads.
```
# Start the server...
/usr/bin/netserver
# Start netperf to connect
/usr/bin/netperf -H <IP_ADDR_OF_SERVER> -t TCP_STREAM -A 16K,16K -l 3600
# Using webbrowser.
firefox http://www.intomail.net/stream.php
```

Select the pressure environment according to the requirements, and then perform the delay test.

## Some tips for using RTLinux

Suppress console messages and prevent memory overuse

```
#The kernel can be started with the kernel parameter quiet, or suppressed after startup, as follows:
echo 1 > /proc/sys/kernel/printk
# Disable Memory overuse to avoid out-of-memory Killer delays
echo 2 > /proc/sys/vm/overcommit_memory
```

### Don't use a desktop or use a lightweight desktop

For better real-time, we do not recommend using a system with a desktop, as this presents a significant challenge for CPU latency. Recommend using minimal Ubuntu, their own QT program, etc.

### Specific cores deal with specific things

The events with high real-time requirements are fixed to a certain core for processing, while the system and other events with low real-time requirements are centralized to a core for processing. Events such as specific interrupts, real-time programs, and so on can be served by a dedicated core.

### MCU using chips

For events requiring higher real-time requirements, it is recommended to use MCU functions on the chip to achieve better real-time control.

## CAN FD driver
At present, only RTLinux uses CANFD driver. CANFD is compatible with CAN function and CAN format CAN be sent. Because rK3588 chip canFD send data format has problems, so only can function is used.
If you must use CANFD, you need to ensure that the last five digits of the sent message are not all zeros or ones (for example, the last two digits are fixed to 01, etc.), so that canFD can be used normally.

```
#Close the can0 device at the transceiver
ip link set can0 down    

#Set 1Mbps bit rate of arbitration section and 3Mbps dbit rate of data section t the transceiver
ip link set can0 type can bitrate 1000000 dbitrate 3000000 fd on

#Show can0 details
ip -details link show can0

#Open the can0 device at the transceiver
ip link set can0 up

#Perform candump on the receiving end, blocking waiting for messages
candump can0

#canfd format, Execute cansend at the sending end to send the message
cansend can0 123##112345678

#can format, Execute cansend at the sending end to send the message
cansend can0 123#1122334455667788
```
