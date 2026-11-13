# Watchdog Usage

## Introduce [step]

Watchdog is actually a timer. After starting, it will start timing. The system or software needs to communicate with the watchdog (commonly known as dog feeding) within a specified time to reset timing, constantly repeat that, to ensure the system and software is working.

If the dog is not fed within the specified time and the watchdog times out, that means the system or application is trapped in a loop or stuck. At this time, the watchdog sends a reset signal to reset the CPU and get rid of the stuck.

iCore-3568JQ has an internal watchdog and an external watchdog. The model of the external watchdog is PC9202.

## Internal watchdog [step]

The internal watchdog is `/dev/watchdog`, used as follows:
```bash
# Write anything (Except capital letter 'V') into it to enable watchdog, and needs to feed it within every 45s
echo 1 > /dev/watchdog

# Write capital letter V to close it
echo V > /dev/watchdog
```
You can also write a program to control the watchdog:
```c
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
<Callout title="Notice" type="warn">

The internal watchdog will start timing right after the `open` function is called.

</Callout>

Reference documentation: `SDK/RKDocs(docs in linux)/common/watchdog`

## External watchdog [step]

The external watchdog is `/dev/wdt_crl`, used as follows:
```bash
# Write different numbers to enable it and set timeout
# The numbers 0, 1, 2, 3 represent 0.64s, 2.56s, 10.24s and 40.96s respectively

# Enable watchdog and set 10.24s as timeout, needs to feed it within every 10.24s, or write other numbers to change timeout
echo 2 > /dev/wdt_crl
```
The external watchdog can also be controlled by a program, just like the internal one; only the device name and the data written need to be changed.

<Callout title="Notice" type="warn">

1. The external watchdog will not start timing after opening until numbers are written into it.
2. The external watchdog only accepts numbers 0, 1, 2, 3.
3. On the **iCore-3568JQ v1.0** version, the external watchdog cannot be closed by software after it is enabled; the power must be cut off to close it.
4. So if you want to update firmware after the external watchdog is enabled, please press the recovery button, disconnect the power and reconnect to enter loader mode; using the `reboot loader` command will cause the board to be reset by the watchdog.

</Callout>