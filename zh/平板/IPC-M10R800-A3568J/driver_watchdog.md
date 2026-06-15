# Watchdog 使用

## 简介

看门狗（watchdog）实际是一个定时器，启动之后会开始计时，系统或者软件需要在规定时间内与看门狗通信（俗称喂狗）重置计时，如此反复下去，以此来确定系统和软件正常运行。

如果规定时间内没有喂狗，看门狗超时，说明系统或应用陷入循环、卡死，此时看门狗会发出复位信号让主控复位，脱离卡死。

IPC-M10R800-A3568J 带有一个内部看门狗和一个外部看门狗，外部看门狗的型号为：PC9202

## 内部看门狗

内部看门狗的设备名称为`/dev/watchdog`，使用方法如下：
```bash
# 写入任意内容（大写字母‘V’除外），开启看门狗，每 45 秒内需要写入一次（喂狗）
echo 1 > /dev/watchdog

# 写入大写字母 V 关闭看门狗
echo V > /dev/watchdog
```
也可以使用程序来控制：
```C
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <syslog.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

void main() {
	unsigned char food = 0;
	int fd_watchdog = open("/dev/watchdog", O_WRONLY);

	if(fd_watchdog < 0) {
		printf("FAILED to open /dev/watchdog, errno: %d, %s\n", errno, strerror(errno));
		syslog(LOG_WARNING, "FAILED to open /dev/watchdog, errno: %d, %s", errno, strerror(errno));
		exit(1);
	}

	// default timeout is 45s
	sleep(40);
	// feed the watchdog once
	ssize_t eaten = write(fd_watchdog, &food, 1);
	if(eaten != 1) {
		printf("FAILED feeding watchdog\n");
		syslog(LOG_WARNING, "FAILED feeding watchdog");
		exit(1);
	}

	// close the watchdog
	food = 'V';
	write(fd_watchdog, &food, 1);
	close(fd_watchdog);
}
```
**注意事项：** 内部看门狗在使用 open 函数打开后会立刻开始计时

参考文档：SDK/RKDocs(linux 为 docs)/common/watchdog

## 外部看门狗

外部看门狗的设备名称是`/dev/wdt_crl`，使用方法如下：
```bash
# 写入不同数字来开启看门狗并设置时间
# 数字 0，1，2，3 分别表示 0.64s，2.56s，10.24s，40.96s

# 开启并定时 10.24 秒，每 10.24 秒之内要写入一次，也可随时写入不同数字更改时间
echo 2 > /dev/wdt_crl
```
外部看门狗同样可以用程序控制，写法和内部看门狗一样，只需要修改设备名称和写入的内容

**注意事项：**

1. 外部看门狗在使用 open 函数打开后不会开始计时，必须要写入数字才开始
2. 外部看门狗只支持写入数字 0，1，2，3
3. 目前 <font color=red>IPC-M10R800-A3568J v1.0</font> 版本，外部看门狗开启后无法软件关闭，需要断电才能关闭
4. 因此，如果开启外部看门狗之后想要烧写固件，请按住 recovery 断开电源再连接进入 loader 模式，使用 reboot loader 命令进入可能会被看门狗重启