
# 定制 Android 固件

## 前言

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
AND_FW_VER: 
```
