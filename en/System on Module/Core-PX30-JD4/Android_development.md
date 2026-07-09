# Android development

## ADB use

### Preface

ADB (the full name is the Android Debug Bridge) is the command-line debugging tool for Android, and it can complete a variety of functions, such as tracking the system logs, uploading and downloading the files, installing the applications, etc.

### Prepare connection

AIO-PX30-JD4 support two ADB connection modes: USB and network connection.

#### USB modes

* On the development board, enter `Options` -> `Developer Options`, and check the `USB Debugging` option.
* Use a Type-C data cable connect host and OTG port of device.
* PX30 Type-C can select device mode only.
    * `Setting` -> `Connected devices` -> `Connecte to PC`

#### Network modes

+ Network ADB
	- Check the AIO-PX30-JD4 IP address, Use command under the PC:

```
adb connect <IP>
adb shell
```

### ADB Installation under Windows

First, please reference [Install RK USB driver](upgrade_firmware.html#windows) to get the driver ready.

Then go to [http://adbshell.com/download/download-adb-for-windows.html](http://adbshell.com/download/download-adb-for-windows.html) to download `adb.zip`, uncompress it to `C:\adb` to ease later use.

Open a cmd window, input:

```
cd C:\adb
adb shell
```

If everything works, you have entered `adb shell`, and can run all kinds of commands available in device.

### ADB installation on Ubuntu

* Install ADB:

```
sudo apt-get install android-tools-adb
```

* Add device identity:

```
mkdir -p ~/.android
vi ~/.android/adb_usb.ini
# append following line
0x2207
```

* Add udev rule:

```
sudo vi /etc/udev/rules.d/51-android.rules
# append following line:
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```

* Replug the USB cable, or run the following command to make the udev rule work:

```
sudo udevadm control --reload-rules
sudo udevadm trigger
```

* Restart adb daemon:

```
sudo adb kill-server
adb start-server
```

### Frequently-used ADB Commands

#### Connection Manage

List all the connected devices:

```
adb devices
```

If there are multiple devices connected, the device serial number is needed to distinguish:

```
export ANDROID_SERIAL=<device serial number>
adb shell ls
```

ADB can also use the TCP/IP network to connect to device:

```
# Restart ADB on the device side and listen at TCP port 5555.
adb tcpip 5555
# Now, the Type-C cable can be disconnected.
# Connect to the device, whose IP is 192.168.1.100 here.
adb connect 192.168.1.100:5555
# Disconnect the device
adb disconnect 192.168.1.100:5555
```

#### Debugging

**Get System Log (adb logcat)**

* Usage

```
adb logcat [Options] [Label]
```

For example:

```
# View all logs.
adb logcat
# Show only part of the logs.
adb logcat -s WifiStateMachine StateMachine
```

#### Running Commands (adb shell)

##### Get Details of System (adb bugreport)

`adb bugreport` is used for bug report, which contains lots of useful information about system.

For example:

```
adb bugreport
# Save to host, then open it with text editor to view ADB
adb bugreport >bugreport.txt
```

#### Root permission

If TARGET_BUILD_VARIANT is in userdebug mode, you need to run it to get root permission:

```
adb root
```

Switch ADB’s device side to root mode so ADB remount and other commands that require root can succeed.

### Application Manage

#### Install Application (adb install)

* Usage:

```
adb install [Options] Application.apk
```

* Options incldue:

```
-l forward-lock
-r Reinstall application, keeping previous data
-s Install to SD card, instead of internal storage
```

For example:

```
# Install facebook.apk
adb install facebook.apk
# Upgrade twitter.apk
adb install -r twitter.apk
```

If installation is successful, it will prompt "Success"; otherwise, it will fail with following messages:

```
* INSTALL_FAILED_ALREADY_EXISTS: You need to reinstall with the -r parameter at this point.
* INSTALL_FAILED_SIGNATURE_ERROR: The applied signatures may not be the same, and may be due to differences between the release and debug signatures. If you are sure that the APK file signature is ok, you can uninstall the old application by using the [adb uninstall](http://www.t-firefly.com/upload/#adb_uninstall) command.Then install again.
* INSTALL_FAILED_INSUFFICIENT_STORAGE: Insufficient storage space, you need to check the device storage.
```

#### Uninstall Application (adb uninstall)

* Usage:

```
adb uninstall Application package name
```

For example:

```
adb uninstall com.android.chrome
```

The package name of application can be listed with:

```
adb shell pm list packages -f
```

The running result:

```
...
package:/system/app/Bluetooth.apk=com.android.bluetooth
...
```

The apk file is ahead，and the follows is the corresponding package name.

#### Commandline Help (adb help)

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

## Compile Android8.1

### Preparation

Compiling Android requires higher configuration requirements for the machine:

* 64 bits CPU.
* 16GB physical memory + swap memory.
* 100GB the free disk space is used for the build, and the source tree takes up an additional approximately 25GB.

The official recommendation of Ubuntu 14.04 operating system, after testing, Ubuntu 12.04 can also compile and run successfully, just need to meet the hardware and software configuration following [http://source.android.com/source/building.html](http://source.android.com/source/building.html)

Initialization of the compilation environment is referred to [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html).

* Install OpenJDK 8:

```
sudo apt-get install openjdk-8-jdk
```

Tip: Install openjdk-8-jdk would changed the default link to the JDK. We can do as follow:

```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```

To switch JDK versions. SDK will use the JDK path set internally when the default JDK of the operating system cannot be found. Therefore, to enable the same machine to compile Android 5.1 and previous versions, it is more convenient to remove the link:

```
$ sudo /var/lib/dpkg/info/openjdk-8-jdk:amd64.prerm remove
```

* Ubuntu 12.04 package installation:

```
sudo apt-get install git gnupg flex bison gperf build-essential \
zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 \
lzop libssl1.0.0 libssl-dev
```

* Ubuntu 14.04 package installation:

```
sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils \
xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
lib32readline-gplv2-dev gcc-multilib libswitch-perl \
libssl1.0.0 libssl-dev
```

### Download Android8.1 SDK

**Android SDK source package is relatively large, you can obtain the Android8.1 source package through the following ways:** [Download link](http://en.t-firefly.com/doc/download/page/id/63.html#other_206)

* Verify MD5 code after downloading:

```
$md5sum ~/firefly_px30_android8.1_git_20211220.001
$md5sum ~/firefly_px30_android8.1_git_20211220.002
$md5sum ~/firefly_px30_android8.1_git_20211220.003

d8291b39a8be4be2cf7bee4676b53397  firefly_px30_android8.1_git_20211220.001
3f2a0b10ed8b5b85b619e81c71fbfb29  firefly_px30_android8.1_git_20211220.002
7732c2ec7dede98098738b1ffb953993  firefly_px30_android8.1_git_20211220.003
```

After confirmation, you can decompress:

```
cd ~/proj/
7z x ./firefly_px30_android8.1_git_20211220.001 -o AIO-PX30-JD4
cd ./AIO-PX30-JD4
git reset --hard
```

Note: be sure to update the remote warehouse after unpacking.
The following method is updated from gitlab:

```
# Enter SDK root directory
cd ~/proj/AIO-Px30-JD4
# First, clone bundle from gitlab
git clone https://gitlab.com/TeeFirefly/px30-oreo-bundle.git .bundle
# After that, run command below to update SDK
.bundle/update
# Download the code for your own branch
git rebase FETCH_HEAD
```

### AIO-PX30-JD4 Compiling method

#### LVDS displays compilation

```
./FFTools/make.sh  -d px30-firefly-aiojd4-lvds -j8 -l px30_evb-userdebug
./FFTools/mkupdate/mkupdate.sh -l px30_evb-userdebug
```

#### AIO-PX30-JD4 Manually compile

Before compilation, execute the following command to configure the environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

*    Compile the kernel:

```
cd ~/proj/AIO-Px30-JD4/android-81/kernel/
make ARCH=arm64 firefly_defconfig
make -j4 ARCH=arm64 px30-firefly-aiojd4-lvds.img
```

*    Compile the uboot:

```
cd ~/proj/AIO-Px30-JD4/android-81/u-boot/
rm -f *LoaderAll*.bin
make clean
make mrproper
./make.sh evb-px30 -j 4
```

*    Compile the Android：

```
cd ~/proj/AIO-PX30-JD4/android-81/
source FFTools/build.sh
lunch px30_evb-userdebug
make installclean
make -j8
./mkimage.sh
```

### Package into unified firmware - update.img

After compiling, you can package the unified firmware with the official script of Firefly, and execute the following command:

```
./FFTools/mkupdate/mkupdate.sh -l px30_evb-userdebug
```

When the packaging is complete, the unified firmware `PX30_Android8.1.0_LVDS_xxxxxx.img` will be generated under `rockdev/Image-px30_evb/`

`update.img` is also easy to package under Windows, copy the compiled files to `rockdev\Image` directory of AndroidTool, and then run `mkupdate.bat` batch file under rockdev directory to create `update.img` and store it in `rockdev\Image` directory.

### Upgrade partition images

At compile time, `./mkimage.sh` repackage `boot.img` and `system.img` and copy other related image files into the directory (rockdev/Image-px30_firefly_aiojd4/).

The following is a list of image files used by common firmware:

*    boot.img ：Android's initial file image, which initializes and loads the system partition.
*    kernel.img ：The kernel image
*    misc.img ：misc partition image, responsible for initiating mode switching and parameter passing of first-aid mode.
*    parameter.txt ：Partition information for emmc.
*    recovery.img ：Image of first aid mode.
*    resource.img ：resource image, which contains boot image and kernel device tree information.
*    system.img ：Android system partition image, ext4 file system format.
*    trust.img ：sleep to wake up relevant files.
*    MiniLoaderAll.bin ：Loader file.
*    uboot.img ：uboot file.
*    vendor.img : driver library
*    oem.img : media file

Refer to [How to upgrade firmware](upgrade_firmware.html) to burn the partition image file.

If you are using a Windows system, copy the above Image file to `rockdev\Image` directory of AndroidTool (firmware upgrade tool under Windows), and then upgrade the partition Image according to the upgrade document. The advantage is that you can use the default configuration without changing the path of the file.

`update.img` is convenient for firmware release, for end users to upgrade the system. It is generally convenient to use partition images during development.


## Customized Android firmware

### preface

There are two ways to customize Android firmware:

* Change the source code and compile the firmware.
* Tailoring the existing firmware.

In the former method, Android can be customized from all levels, with great freedom, but high requirements for the compilation environment and technology, which can be referred to Compile Android8.1 firmware].

Now introduce the latter method, which is divided into three phases: unpacking, customization and packaging. The host operating system is Linux, and the tools used are open source software.

## Firmware format

Unified firmware `release_update.img` contains the boot loader `loader.img` and the actual firmware data - `update.img`.

```
release_update.img
|- loader.img
`- update.img
```

`update.img` is a composite file containing multiple files described by package-file. A typical package-file is:

```
# NAME    Relative path
#
# HWDEF   HWDEF
package-file package-file
bootloader Image/MiniLoaderAll.bin
parameter  Image/parameter.txt
trust      Image/trust.img
uboot      Image/uboot.img
misc       Image/misc.img
resource   Image/resource.img
kernel     Image/kernel.img
boot    Image/boot.img
recovery   Image/recovery.img
system     Image/system.img
backup     RESERVED
update-script update-script
recover-script recover-script
```

* package-file
	* `update.img` package description file, `update.img` also contains a `package-file`.
* Image/MiniLoaderAll.bin
	* Start the loader - bootloader.
* Image/parameter.txt
	* Parameter file, can set the kernel boot parameters, there are important partition information.
* Image/trust.img
	* `trust.img` is the U-Boot as the secondary loader package.
* Image/misc.img
	* Image of misc partition, used to control whether Android starts normally or enters Recovery Mode.
* Image/kernel.img
	* Android kernel.
* Image/resource.img
	* Resource image with kernel boot image and kernel Device Tree Blob.
* Image/boot.img
	* The memory boot disk (initrd) of the Android kernel, the first root file system to be loaded after the kernel is started, contains important initialization actions that generally do not need to be changed.
* Image/recovery.img
	* Android image,including Recovery Mode with kernel and root file system of Recovery Mode.
* Image/system.img
	* Corresponds to the Android `/system` partition and is the following custom object.

Unpacking, is to extract the `update.img` from `release_update.img`, then the extracted with `package-file` statement by multiple files.

Packaging is an inverse process to synthesize the multiple files listed in the package-file to `update.img`, add to the `loader.img`, and finally generate the `release_update.img`.

### Tools preparation

```
git clone https://github.com/TeeFirefly/rk2918_tools.git
cd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```

### Unpack

* Extract the `release_update.img`

```
$ cd /path/to/your/firmware/dir
$ img_unpack PX30_Android8.1.0_LVDS_xxxxxx.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```

* Extract the `update.img`

```
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x000002AC
Image/MiniLoaderAll.bin	0x00001000	0x0004394E
Image/parameter.txt	0x00045000	0x00000368
Image/trust.img	0x00045800	0x00400000
Image/uboot.img	0x00445800	0x00400000
Image/misc.img	0x00845800	0x0000C000
Image/resource.img	0x00851800	0x00038800
Image/kernel.img	0x0088A000	0x012D2014
Image/boot.img	0x01B5C800	0x0017893C
Image/recovery.img	0x01CD5800	0x00866B8C
Image/system.img	0x0253C800	0x41A9110C
Image/vendor.img	0x43FCE000	0x1206E0A0
Image/oem.img	0x5603C800	0x00058094
RESERVED	0x00000000	0x00000000
UnPack OK!
```

* View the file tree under the update directory

```
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── MiniLoaderAll.bin
│   ├── misc.img
│   ├── oem.img
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   ├── uboot.img
│   └── vendor.img
├── loader.img
├── package-file
└── RESERVED

1 directory, 15 files
```

Now that the firmware has been unpacked successfully, let's start customizing it.

#### Pack

First, check the size of `system.img` and make necessary size adjustments against the partition of the `parameter` file.(Refer to the documentation [The file format parameter](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf)

For example, the system partition size in `parameter.txt` file, you can find the `CMDLINE` line and then find the `system` string:

```
0x00200000@0x000B0000(system)
```

Before @ is the size of the partition, the unit is 512 bytes, so the size of the system partition is:

```
$ echo $(( 0x00200000 * 512 / 1024 / 1024))M
1024M
```

As long as the size of `system.img` does not exceed 1024M, the parameter file does not need to be changed.

If the partition does not change, you can use the upgrade tool to upgrade the new `system.img` directly to the `system` partition on the development board for testing. Otherwise, you need to make new firmware and burn it before testing.

Here are the steps required to package it into a unified firmware `update.img`:

* Compose update.img:

```
# The current directory is still update/, with package-file and all files listed in package-file.
# Copy the parameter file to paramter, because it is used by default by afptool.
$ cp Image/parameter.txt parameter
$ afptool -pack . ../update_new.img
------ PACKAGE ------
Add file: ./package-file
Add file: ./Image/MiniLoaderAll.bin
Add file: ./Image/parameter.txt
Add file: ./Image/trust.img
Add file: ./Image/uboot.img
Add file: ./Image/misc.img
Add file: ./Image/resource.img
Add file: ./Image/kernel.img
Add file: ./Image/boot.img
Add file: ./Image/recovery.img
Add file: ./Image/system.img
Add file: ./Image/vendor.img
Add file: ./Image/oem.img
Add file: ./RESERVED
Add CRC...
------ OK ------
Pack OK!
```


* Compose release_update.img ：

```
$ img_maker -rk33 loader.img update_new.img release_update_new.img
generate image...
append md5sum...
success!
```

`release_update_new.img` is a unified firmware file that can be upgraded for the final generation

### FAQS

**Q1 :** Where's the firmware version Settings?

**A1 :** In the parameter file, you can find the downward direction and modify it. Note that the version number is a number, and the middle two dots cannot be omitted.

```
FIRMWARE_VER: 8.1
```
