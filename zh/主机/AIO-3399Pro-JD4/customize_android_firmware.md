
# 定制 Android 固件

## 前言

**注意**： 以下内容仅适用于 Android8.1 及以下版本。

定制 Android 固件，有两种方法：

* 改源码，然后编译生成固件。
* 在现有固件的基础上进行裁剪。

前一种方法，可以从各个层面去定制 Android，自由度大，但对编译环境和技术要求比较高，参见[《编译 Android 固件》](compile_android7.1_industry_firmware.md)一文。 现在介绍后一种方法，分为解包、定制和打包三个阶段。主机操作系统为 Linux，采用的工具为开源软件。

## 固件格式

统一固件 `release_update.img`，内含启动加载器 `loader.img` 和真正的固件数据 `update.img`

```
release_update.img
|- loader.img
`- update.img
```

`update.img` 是个复合文件，内含多个文件，由 `package-file` 描述。一个典型的 `package-file` 为：

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
	* `update.img` 的打包说明文件，`update.img` 里也含有一份 `package-file`。
* Image/MiniLoaderAll.bin
	* 启动加载器，即 bootloader。
* Image/parameter.txt
	* 参数文件，可以设定内核启动参数，里面有重要的分区信息。
* Image/trust.img
	* `trust.img` 是 U-Boot 作为二级 loader 的打包。
* Image/misc.img
	* misc 分区的映像，用来控制 Android 是正常启动，还是进入急救模式（Recovery Mode）。
* Image/kernel.img
	* Android 内核。
* Image/resource.img
	* 资源映像，内有内核开机图片和内核设备树信息 (Device Tree Blob)。
* Image/boot.img
	* Android 内核的内存启动盘 (initrd)，是内核启动后最先加载的根文件系统，包含重要的初始化动作，一般不需要改动。
* Image/recovery.img
	* Android 急救模式的映像，内含内核和急救模式的根文件系统。
* Image/system.img
	* 对应于 Android 的 `/system` 分区，是以下的定制对象。

解包，就是提取 `release_update.img` 里的 `update.img`, 然后解压出内含 `package-file` 所声明的多个文件。

打包，则是个逆过程，将 `package-file` 将所列的多个文件合成 `update.img`，加进 `loader.img`，最终生成 `release_update.img` 。

## 工具准备

```
git clone https://github.com/TeeFirefly/rk2918_tools.git
cd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```

## 解包




* 解压 release_update.img

```
$ cd /path/to/your/firmware/dir
$ img_unpack AIO-3399ProJD4_Android8.1.0_HDMI_190509.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```

* 解压 update.img

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

* 查看 update 目录下的文件树

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

这样，固件就解包成功了，下面开始定制。


### 打包

首先要检查一下 `system.img` 的大小，对照 `parameter` 文件的分区情况(可参考文档 [Parameter 文件格式](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf)，作必要的大小调整。

例如 `parameter.txt` 文件里的 `system` 分区大小，可以找到 `CMDLINE` 一行，然后找到 `system` 字符串：

```
0x00200000@0x000B0000(system)
```

@ 前面就是分区的大小，单位是 512 字节，这样该 system 分区的大小就是：

```
$ echo $(( 0x00200000 * 512 / 1024 / 1024))M
1024M
```

只要 `system.img` 的大小不超过 1024M，`parameter` 文件就不用更改。

如果分区不用更改，可以直接用烧写工具将新的 `system.img` 烧写到开发板的 `system` 分区上做试验。否则，需要制作新固件并烧写后再行测试。

以下是打包成统一固件 `update.img` 所需要的步骤：

* 合成 update.img ：



```
# 当前的目录仍然为 update/ ，内有 package-file, package-file 所列的文件均存在
# 将参数文件拷贝一份到 paramter, 因为 afptool 默认要用到
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

* 合成 release_update.img ：

```
$ img_maker -rk33 loader.img update_new.img release_update_new.img
generate image...
append md5sum...
success!
```

`release_update_new.img` 即为最终生成的可烧写的统一固件文件。

## 常见问题

### 固件的版本在哪设置

在 `parameter` 文件中找到下行并修改即可，注意版本号为数字，中间两个点号不能省略。

```
AND_FW_VER: 8.1
```