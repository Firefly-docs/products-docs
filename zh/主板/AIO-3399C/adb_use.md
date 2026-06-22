# ADB 使用

## 前言

ADB，全称 Android Debug Bridge，是 Android 的命令行调试工具，可以完成多种功能，如跟踪系统日志，上传下载文件，安装应用等。


## 准备连接

### USB 数据线 ADB

* AIO-3399C 用 Type-C 数据线连接设备 Type-C 口和主机
* AIO-3399C(AI) 用双端公头 USB 数据线连接设备蓝色 USB 3.0 口和主机，然后根据当前Android版本勾选对应路径下的Connect to PC
   * Android7.1、Android8.1 选择 Setting -> USB，然后勾选 Connect to PC
   * Android10.0 选择 Setting -> Connected devices 然后勾选 Connect to PC

![](../../../rk3399_img/AIO-3399C/adb_connection.jpg)

当设备端状态栏提示 `USB debugging connected` 时，便可进行调试：

```
adb devices
adb shell
```


## 网络 ADB

* `设置` -> `开发者选项` -> `网络 ADB 调试`

查看开发板 IP 地址，PC 端通过网络访问：

```
adb connect + IP
adb shell
```

## Windows下的 ADB 安装

首先参照[安装 RK USB 驱动](03-upgrade_firmware.html#an-zhuang-rk-usb-qu-dong)一节安装好驱动。

然后到 [http://www.t-firefly.com/share/index/index/id/6afafdf7c54b8eb95e81156d94e54db5.html](http://www.t-firefly.com/share/index/index/id/6afafdf7c54b8eb95e81156d94e54db5.html) 下载 `adb.zip`，解压到 `C:\adb` 以方便调用。

打开命令行窗口，输入：

```
cd C:\adb
adb shell
```

如果一切正常，就可以进入 `adb shell`，在设备上面运行命令。

## Ubuntu 下的 ADB 安装

* 安装 ADB 工具：

```
sudo apt-get install android-tools-adb
```

* 加入设备标识：

```
mkdir -p ~/.android
vi ~/.android/adb_usb.ini
# 添加以下一行
0x2207
```

* 加入 udev 规则：

```
sudo vi /etc/udev/rules.d/51-android.rules
# 添加以下一行：
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```

* 重新插拔 USB 线，或运行以下命令，让 udev 规则生效：

```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

* 重新启动 ADB 服务器

```
sudo adb kill-server
adb start-server
```

## 常用 ADB 命令

### 连接管理

列出所有连接设备及其序列号：

```
adb devices
```

如果有多个连接设备，则需要使用序列号来区分：

```
export ANDROID_SERIAL=<设备序列号>
adb shell ls
```

多设备下连接指定设备

```
adb -s 序列号 shell
```

可以通过网络来连接 ADB：

```
# 让设备端的 adbd 重启，并在 TCP 端口 5555 处监听
adb tcpip 5555
# 此时可以断开 USB 连接
# 远程连接设备，设备的 IP 地址是 192.168.1.100
adb connect 192.168.1.100:5555
# 断开连接
adb disconnect 192.168.1.100:5555
```

## 调试

### 获取系统日志 adb logcat

* 用法

```
adb logcat [选项] [应用标签]
```

* 示例

```
# 查看全部日志
adb logcat
# 仅查看部分日志
adb logcat -s WifiStateMachine StateMachine
```

### 运行命令 adb shell

#### 获取详细运行信息 adb bugreport

`adb bugreport` 用于错误报告，里面包含大量有用的信息。

* 示例

```
adb bugreport
# 保存到本地，方便用编辑器查看
adb bugreport >bugreport.txt
```

### root 权限

如果 TARGET_BUILD_VARIANT 使用的是 userdebug 模式，要获得 root 权限，需要先运行：

```
adb root
```

让 ADB 的设备端切换到 root 权限模式，这样 `adb remount` 等需要 root 权限的命令才会成功。

## 应用管理

### 安装应用 adb install

* 用法：

```
adb install [选项] 应用包.apk
```

选项包括：

```
-l forward-lock
-r 重新安装应用，保留原先数据
-s 安装到 SD 卡上，而不是内部存储
```

* 示例：

```
# 安装 facebook.apk
adb install facebook.apk
# 升级 twitter.apk
adb install -r twitter.apk
```

如果安装成功，工具会返回成功提示 `Success`；失败的话，一般是以下几种情况:

* INSTALL_FAILED_ALREADY_EXISTS: 此时需要用 `-r` 参数来重新安装。
* INSTALL_FAILED_SIGNATURE_ERROR: 应用的签名不一致，可能是发布版和调试版签名不同所致。如果确认 APK 文件签名正常，可以用 `adb uninstall` 命令先卸载旧的应用，然后再安装。
* INSTALL_FAILED_INSUFFICIENT_STORAGE: 存储空间不足，需要检查设备存储情况。

### 卸载应用 adb uninstall

* 用法：

```
adb uninstall 应用包名称
```

* 示例：

```
adb uninstall com.android.chrome
```

应用包名称可以用以下命令列出：

```
adb shell pm list packages -f
```

运行结果是：

```
package:/system/app/Bluetooth.apk=com.android.bluetooth
```

前面是 apk 文件，后面则是对应的包名称。

### 命令行帮助信息 adb help

```
Android Debug Bridge version 1.0.31

 -a                            - directs adb to listen on all interfaces for a connection
 -d                            - directs command to the only connected USB device
                                 returns an error if more than one USB device is present.
 -e                            - directs command to the only running emulator.
                                 returns an error if more than one emulator is running.
 -s <specific device>          - directs command to the device or emulator with the given
                                 serial number or qualifier. Overrides ANDROID_SERIAL
                                 environment variable.
 -p <product name or path>     - simple product name like 'sooner', or
                                 a relative/absolute path to a product
                                 out directory like 'out/target/product/sooner'.
                                 If -p is not specified, the ANDROID_PRODUCT_OUT
                                 environment variable is used, which must
                                 be an absolute path.
 -H                            - Name of adb server host (default: localhost)
 -P                            - Port of adb server (default: 5037)
 devices [-l]                  - list all connected devices
                                 ('-l' will also list device qualifiers)
 connect <host>[:<port>]       - connect to a device via TCP/IP
                                 Port 5555 is used by default if no port number is specified.
 disconnect [<host>[:<port>]]  - disconnect from a TCP/IP device.
                                 Port 5555 is used by default if no port number is specified.
                                 Using this command with no additional arguments
                                 will disconnect from all connected TCP/IP devices.

device commands:
  adb push [-p] <local> <remote>
                               - copy file/dir to device
                                 ('-p' to display the transfer progress)
  adb pull [-p] [-a] <remote> [<local>]
                               - copy file/dir from device
                                 ('-p' to display the transfer progress)
                                 ('-a' means copy timestamp and mode)
  adb sync [ <directory> ]     - copy host->device only if changed
                                 (-l means list but don't copy)
                                 (see 'adb help all')
  adb shell                    - run remote shell interactively
  adb shell <command>          - run remote shell command
  adb emu <command>            - run emulator console command
  adb logcat [ <filter-spec> ] - View device log
  adb forward --list           - list all forward socket connections.
                                 the format is a list of lines with the following format:
                                    <serial> " " <local> " " <remote> "\n"
  adb forward <local> <remote> - forward socket connections
                                 forward specs are one of:
                                   tcp:<port>
                                   localabstract:<unix domain socket name>
                                   localreserved:<unix domain socket name>
                                   localfilesystem:<unix domain socket name>
                                   dev:<character device name>
                                   jdwp:<process pid> (remote only)
  adb forward --no-rebind <local> <remote>
                               - same as 'adb forward <local> <remote>' but fails
                                 if <local> is already forwarded
  adb forward --remove <local> - remove a specific forward socket connection
  adb forward --remove-all     - remove all forward socket connections
  adb jdwp                     - list PIDs of processes hosting a JDWP transport
  adb install [-l] [-r] [-d] [-s] [--algo <algorithm name> --key <hex-encoded key> --iv <hex-encoded iv>] <file>
                               - push this package file to the device and install it
                                 ('-l' means forward-lock the app)
                                 ('-r' means reinstall the app, keeping its data)
                                 ('-d' means allow version code downgrade)
                                 ('-s' means install on SD card instead of internal storage)
                                 ('--algo', '--key', and '--iv' mean the file is encrypted already)
  adb uninstall [-k] <package> - remove this app package from the device
                                 ('-k' means keep the data and cache directories)
  adb bugreport                - return all information from the device
                                 that should be included in a bug report.

  adb backup [-f <file>] [-apk|-noapk] [-obb|-noobb] [-shared|-noshared] [-all] [-system|-nosystem] [<packages...>]
                               - write an archive of the device's data to <file>.
                                 If no -f option is supplied then the data is written
                                 to "backup.ab" in the current directory.
                                 (-apk|-noapk enable/disable backup of the .apks themselves
                                    in the archive; the default is noapk.)
                                 (-obb|-noobb enable/disable backup of any installed apk expansion
                                    (aka .obb) files associated with each application; the default
                                    is noobb.)
                                 (-shared|-noshared enable/disable backup of the device's
                                    shared storage / SD card contents; the default is noshared.)
                                 (-all means to back up all installed applications)
                                 (-system|-nosystem toggles whether -all automatically includes
                                    system applications; the default is to include system apps)
                                 (<packages...> is the list of applications to be backed up.  If
                                    the -all or -shared flags are passed, then the package
                                    list is optional.  Applications explicitly given on the
                                    command line will be included even if -nosystem would
                                    ordinarily cause them to be omitted.)

  adb restore <file>           - restore device contents from the <file> backup archive

  adb help                     - show this help message
  adb version                  - show version num

scripting:
  adb wait-for-device          - block until device is online
  adb start-server             - ensure that there is a server running
  adb kill-server              - kill the server if it is running
  adb get-state                - prints: offline | bootloader | device
  adb get-serialno             - prints: <serial-number>
  adb get-devpath              - prints: <device-path>
  adb status-window            - continuously print device status for a specified device
  adb remount                  - remounts the /system partition on the device read-write
  adb reboot [bootloader|recovery] - reboots the device, optionally into the bootloader or recovery program
  adb reboot-bootloader        - reboots the device into the bootloader
  adb root                     - restarts the adbd daemon with root permissions
  adb usb                      - restarts the adbd daemon listening on USB
  adb tcpip <port>             - restarts the adbd daemon listening on TCP on the specified port
networking:
  adb ppp <tty> [parameters]   - Run PPP over USB.
 Note: you should not automatically start a PPP connection.
 <tty> refers to the tty for PPP stream. Eg. dev:/dev/omap_csmi_tty1
 [parameters] - Eg. defaultroute debug dump local notty usepeerdns

adb sync notes: adb sync [ <directory> ]
  <localdir> can be interpreted in several ways:

  - If <directory> is not specified, both /system and /data partitions will be updated.

  - If it is "system" or "data", only the corresponding partition
    is updated.

environmental variables:
  ADB_TRACE                    - Print debug information. A comma separated list of the following values
                                 1 or all, adb, sockets, packets, rwx, usb, sync, sysdeps, transport, jdwp
  ANDROID_SERIAL               - The serial number to connect to. -s takes priority over this if given.
  ANDROID_LOG_TAGS             - When used with the logcat option, only these debug tags are printed.
```