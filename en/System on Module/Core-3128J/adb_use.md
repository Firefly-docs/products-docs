# ADB Usage

## Introduction

Adb( the full name is the Android Debug Bridge) is the command-line debugging tool for Android, and it can complete a variety of functions, such as tracking the system logs, uploading and downloading the files, installing the applications, etc.

## Preparing to connect

In Android system, go to "Settings" -> "Development options", check the "USB Debugging" option.  
Use a micro USB OTG cable to connnect development board and host.

## ADB installation for Windows

First, please reference Install RK USB driver to get the  driver ready.  
Then download [adb.zip](https://drive.google.com/file/d/1ySJ4De-Lmo3MhsXLfGc28yfXihW3ahtP/view?usp=sharing), and uncompress it to `C:\adb` to ease later use.
Open a cmd window, input:  

```
cd C:\adb
adb shell
```

If everything works out, you can enter the `adb shell` and run the command on the device. 

## Adb Installation for Ubuntu

* Install the adb tool:

```
sudo apt-get install android-tools-adb
```

* Add the device id:

```
mkdir -p ~/.android
vi ~/.android/adb_usb.ini
# append following line
0x2207
```

* Add the udev rule:

```
sudo vi /etc/udev/rules.d/51-android.rules
# append following line:
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```

* Replug the USB cable or run the following command so that the udev rule can take effect:

```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

*  Restart the adb server:

```
sudo adb kill-server
adb start-server
```

## Common Adb Commands

### Connection Management

List all connected devices and their serial numbers:

```
adb devices
```

If there are more than one connection device, the serial numbers will be required to distinguish them:

```
export ANDROID_SERIAL=<device serial no>
adb shell ls
```

The adb can be connected through the network:
```
# Restart adbd in device, which will be listened at TCP port 5555.
adb tcpip 5555
# Now, the USB cable can be disconnected.
# Connect to the device, whose IP is 192.168.1.100 here.
adb connect 192.168.1.100:5555
# Disconnect the device
adb disconnect 192.168.1.100:5555
```

### Debug

#### Get the system logs adb logcat

* Usage

```
adb logcat [Options] [Label]
```

* Example:

```
# Show all the log.
adb logcat
# Show only part of the log matching tags specified.
adb logcat -s WifiStateMachine StateMachine
```

#### Run the command adb shell

#### Get the detailed running information adb bugreport

`adb bugreport` is used for bug report, which contains lots of useful information about system.  

* Example:

```
adb bugreport
# Save to host, then open it with text editor to view adb
adb bugreport >bugreport.txt
```
#### root privilege

If the userdebug mode is used for TARGET_BUILD_VARIANT, you need to run the following command so as to get the root privilege first:

```
adb root
```

Allow the device end of adb to switch to root privilege mode so that commands such as adb remount that require root privileges will succeed.

#### Application Management

##### Install the application adb install

* Usage：

```
adb install [Options] Application.apk
```

* The options include:

```
-l forward-lock
-r Reinstall application, keeping previous data
-s Install to SD card, instead of internal storage
```

* Example:

```
# Install facebook.apk
adb install facebook.apk
# Upgrade twitter.apk
adb install -r twitter.apk
```

If the installation is successful, the tool will return a success prompt “Success”, and if the installation is unsuccessful, in general, it is generally caused by the following conditions:

* INSTALL_FAILED_ALREADY_EXISTS: "-r" parameter is needed to reinstall.
* INSTALL_FAILED_SIGNATURE_ERROR: The signatures do not match. It sometimes happens that the released application and the debug one have different signatures. If the signature of APK is confirmed to be valid, use [adb uninstall](http://www.t-firefly.com/upload/#adb_uninstall) to uninstall old application first, then try again to install.
* INSTALL_FAILED_INSUFFICIENT_STORAGE: The storage space is not enough, need to check. Install the SD card or uninstall some unused applications to free some space.

##### Uninstall the application adb uninstall

* Usage:

```
adb uninstall ApplicationPackageName
```

* Example:

```
adb uninstall com.android.chrome
```
The application package name can be listed with the following command:

* The running result is:

```
package:/system/app/Bluetooth.apk=com.android.bluetooth
```

It is the apk file in the front, followed by the corresponding package name.

#### Command line help information adb help

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