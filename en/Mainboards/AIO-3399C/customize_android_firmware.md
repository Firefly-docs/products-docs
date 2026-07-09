
# Customized Android firmware

## preface

**Note**: The following is only compatible with Android 8.1 and below.

There are two ways to customize Android firmware:

* Change the source code and compile the firmware.
* Tailoring the existing firmware.

In the former method, Android can be customized from all levels, with great freedom, but high requirements for the compilation environment and technology, which can be referred to [Compile Android firmware].(compile_android8.1_firmware.html). Now introduce the latter method, which is divided into three phases: unpacking, customization and packaging. The host operating system is Linux, and the tools used are open source software.

## Firmware format

Unified firmware `release_update.img` contains the boot loader `loader.img` and the actual firmware data - `update.img`.

```
release_update.img
|- loader.img
`- update.img
```

`update.img` is a composite file containing multiple files described by package-file. A typical package-file is:

```
# NAME Relative path package-file package-file bootloader Image/MiniLoaderAll.bin parameter
Image/parameter.txt trust
Image/trust.img uboot
Image/uboot.img misc
Image/misc.img resource
Image/resource.img kernel
Image/kernel.img boot
Image/boot.img recovery
Image/recovery.img system
Image/system.img backup RESERVED
#update-script update-script #recover-script recover-script
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

## Tools preparation

```
git clone https://github.com/TeeFirefly/rk2918_tools.git
cd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```

## Unpack

* Extract the `release_update.img`

```
$ cd /path/to/your/firmware/dir
$ img_unpack Firefly-RK3399_20161027.img img
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
package-file	0x00000800	0x00000280
Image/MiniLoaderAll.bin	0x00001000	0x0003E94E
Image/parameter.txt	0x00040000	0x00000350
Image/trust.img	0x00040800	0x00400000
Image/uboot.img	0x00440800	0x00400000
Image/misc.img	0x00840800	0x0000C000
Image/resource.img	0x0084C800	0x0003FE00
Image/kernel.img	0x0088C800	0x00F5D00C
Image/boot.img	0x017EA000	0x0014AD24
Image/recovery.img	0x01935000	0x013C0000
Image/system.img	0x02CF5000	0x2622A000
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
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   └── uboot.img
├── package-file
└── RESERVED

1 directory, 12 files
```




Now that the firmware has been unpacked successfully, let's start customizing it.

## Customize

### Customize the system.img

`System.img` is an ext4 file format image file that can be mounted directly to the system for modification:

```
sudo mkdir -p /mnt/system
sudo mount -o loop Image/system.img /mnt/system
cd /mnt/system
# Notice how much space is left when modify inside,and don't add too much APK.
# After modification, uninstall.
cd /
sudo umount /mnt/system
```

**Note:** The remaining space of the `system.img` is basically 0. If the file is not deleted, the capacity of `system.img` needs to be expanded, and the partition Settings in the parameter file are adjusted accordingly according to the final file size. The following is an example of how to extend the space. Before extending, run mount to see how the system is mounted and ensure that system.img has been unmounted:

```
# Increase the space by 128M
dd if=/dev/zero bs=1M count=128 >> Image/system.img
# Extend file system information
e2fsck -f Image/system.img
resize2fs Image/system.img
```

### Pack

First, check the size of `system.img` and make necessary size adjustments against the partition of the `parameter` file.(Refer to the documentation [The file format parameter](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf))

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

## FAQS

**Q1 :** Where's the firmware version Settings?

**A1 :** In the parameter file, you can find the downward direction and modify it. Note that the version number is a number, and the middle two dots cannot be omitted.

```
AND_FW_VER: 7.1.1
```