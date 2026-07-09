# Customized Android firmware

## preface

There are two ways to customize Android firmware:

* Change the source code and compile the firmware.
* Tailoring the existing firmware.

In the former method, Android can be customized from all levels, with great freedom, but high requirements for the compilation environment and technology, which can be referred to [Compile Android firmware](compile_android_firmware.md). Now introduce the latter method, which is divided into three phases: unpacking, customization and packaging. The host operating system is Linux, and the tools used are open source software.

## Firmware format

Unified firmware `release_update.img` contains the boot loader `loader.img` and the actual firmware data - `update.img`.

```bash
release_update.img
|- loader.img
`- update.img
```

`update.img` is a composite file containing multiple files described by package-file. A typical package-file is:

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      RK3288Loader_uboot_Apr212014_134842.bin
parameter       rk3288-3.10-uboot-data1G.parameter.txt
uboot           Image/uboot.img
misc            Image/misc.img
resource        Image/resource.img
kernel          Image/kernel.img
boot            Image/boot.img
recovery        Image/recovery.img
system          Image/system.img
backup          RESERVED
update-script   update-script
recover-script  recover-script
```

* package-file
    + Packing instruction file for the update.img, a package-file is also contained in the update.img.
* RK3288Loader_uboot_Apr212014_134842.bin
    + Boot loader(i.e. bootloader).
* rk3288-3.10-uboot-data1G.parameter.txt
    + Parameter file, which can be used to set the kernel startup parameters, and there are important partition information in it.
* Image/misc.img
    + Image of the partition, which is used to control whether the Android starts normally or enters the emergency mode(Recovery Mode).
* Image/kernel.img
    + Android kernel.
* Image/resource.img
    + Resource image, which contains the boot image and the device tree blob information of the kernel.
* Image/boot.img
    + The memory boot disk (initrd) of the Android kernel, which is the root file system that is loaded first after the kernel startup and contains important initialization actions, in general, the change is not required.
* Image/recovery.img
    + Image in the emergency mode for Android, which contains the root file system of the kernel and emergency modes.
* Image/system.img
    + Corresponding to the /system partition of Android, and it is the following custom object.

Unpacking, is to extract the `update.img` from `release_update.img`, then the extracted with `package-file` statement by multiple files.

Packaging is an inverse process to synthesize the multiple files listed in the package-file to `update.img`, add to the `loader.img`, and finally generate the `release_update.img`.

## Tools preparation

```bash
git clone https://github.com/TeeFirefly/rk2918_tools.git
cd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```

## Unpack

* Extract the `release_update.img`

```bash
$ cd /path/to/your/firmware/dir
$ img_unpack Firefly-RK3288_Android4.4_20140818.img img
rom version: 4.4.2
build time: 2014-08-18 14:25:57
chip: 80
checking md5sum....OK
```

* Extract the `update.img`

```bash
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x00000285
RK3288Loader_uboot_Apr212014_134842.bin	0x00001000	0x0004694E
rk3288-3.10-uboot-data1G.parameter.txt	0x00048000	0x000005A1
Image/misc.img	0x00048800	0x0000C000
Image/kernel.img	0x00055000	0x00578E3C
Image/resource.img	0x005CE000	0x0001C400
Image/boot.img	0x005EA800	0x0011F6CF
Image/recovery.img	0x0070A000	0x0040F6AE
Image/system.img	0x00B19800	0x180EF000
RESERVED	0x00000000	0x00000000
update-script	0x18C09000	0x000003A5
recover-script	0x18C09800	0x0000010A
UnPack OK!
```

* View the file tree under the update directory

```bash
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── misc.img
│   ├── recovery.img
│   ├── resource.img
│   ├── uboot.img
│   └── system.img
├── package-file
├── recover-script
├── RESERVED
├── rk3288-3.10-uboot-data1G.parameter.txt
├── RK3288Loader_uboot_Apr212014_134842.bin
└── update-script
```

Now that the firmware has been unpacked successfully, let's start customizing it.

## Customize

### Customize the system.img

`System.img` is an ext4 file format image file that can be mounted directly to the system for modification:

```bash
sudo mkdir -p /mnt/system
sudo mount -o loop Image/system.img /mnt/system
cd /mnt/system
# Notice how much space is left when modify inside,and don't add too much APK.
# After modification, uninstall.
cd /
sudo umount /mnt/system
```

**Note:** The remaining space of the `system.img` is basically 0. If the file is not deleted, the capacity of `system.img` needs to be expanded, and the partition Settings in the parameter file are adjusted accordingly according to the final file size. The following is an example of how to extend the space. Before extending, run mount to see how the system is mounted and ensure that system.img has been unmounted:

```bash
# Increase the space by 128M
dd if=/dev/zero bs=1M count=128 >> Image/system.img
# Extend file system information
e2fsck -f Image/system.img
resize2fs Image/system.img
```

### Pack

First, check the size of `system.img` and make necessary size adjustments against the partition of the `parameter` file.(Refer to the documentation [The file format parameter](http://www.t-firefly.com/download/firefly-rk3288/rockchip/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf).

For example, the system partition size in `parameter.txt` file, you can find the `CMDLINE` line and then find the `system` string:

```bash
0x00180000@0x00092000(system)
```

Before @ is the size of the partition, the unit is 512 bytes, so the size of the system partition is:

```bash
$ echo $(( 0x00200000 * 512 / 1024 / 1024))M
1024M
```

As long as the size of `system.img` does not exceed 768M, the parameter file does not need to be changed.

If the partition does not change, you can use the upgrade tool to upgrade the new `system.img` directly to the `system` partition on the development board for testing. Otherwise, you need to make new firmware and burn it before testing.

Here are the steps required to package it into a unified firmware `update.img`:

* Compose update.img:

```bash
# The current directory is still update/, which contains the package-file, and the files listed by the package-file exist
# Copy the parameter file to the paramter, by default, the afptool is used to
$ cp rk3288-3.10-uboot-data1G.parameter.txt parameter
$ afptool -pack . ../update_new.img
------ PACKAGE ------
Add file: ./package-file
Add file: ./RK3288Loader_uboot_Apr212014_134842.bin
Add file: ./rk3288-3.10-uboot-data1G.parameter.txt
Add file: ./Image/uboot.img
Add file: ./Image/misc.img
Add file: ./Image/kernel.img
Add file: ./Image/resource.img
Add file: ./Image/boot.img
Add file: ./Image/recovery.img
Add file: ./Image/system.img
Add file: ./RESERVED
Add file: ./update-script
Add file: ./recover-script
Add CRC...
------ OK ------
Pack OK!
```

* Compose release_update.img ：

```bash
$ img_maker -rk32 loader.img update_new.img release_update_new.img
generate image...
append md5sum...
success!
```

`release_update_new.img` is a unified firmware file that can be upgraded for the final generation

## FAQS

**Q1 :** Where's the firmware version Settings?

**A1 :** In the parameter file, you can find the downward direction and modify it. Note that the version number is a number, and the middle two dots cannot be omitted.

```bash
FIRMWARE_VER:5.1.0
```