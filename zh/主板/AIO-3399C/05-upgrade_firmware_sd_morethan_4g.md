# 若固件大于4G的SD升级卡制作教程:

因为RK提供的`SD_Firmware_Tool`升级工具，制作的升级分区是FAT32格式的，所以支持的固件大小最大为4G，现要将该FAT32格式分区重新格式化为EXT4格式。<font color = "red">（注：前面的工作和正常的做SD升级卡无异，但是最后需要重新格式化分区和放入升级所需要的文件）</font>

## 准备工作

* AIO-3399C
* 电脑
* MicroSD卡(即TF 卡)
* Type-C数据线
* USB读卡器

<a id="SD_Firmware_Tool"></a>

前往[SD固件工具下载页面](http://www.t-firefly.com/doc/download/page/id/54.html#other_541)下载`SD_Firmware_Tool`并解压。

## 运行`SD_Firmware_Tool.exe`
![](../../../rk3399_img/sdfirmwaretool.zh_CN.jpg)

1. 下载SD卡制作工具：[SD_Firmware_Tool](http://www.t-firefly.com/doc/download/page/id/54.html#other_541)
2. 可修改**config.ini**文件，设置`Selected=1`则语言为中文，设置`Selected=2`则语言为英文。
3. 下载需要升级到主板上的统一固件。
4. 将MicroSD插入USB读卡器，再插入到电脑USB口上。
5. 打开SD_Firmware_Tool，勾选“固件升级”框，选择正确的可移动磁盘设备和升级固件。
6. 点击开始创建之后，等待创建结束。
7. 取出MicroSD卡，插入主板的MicroSD卡插槽，对主板上电开机，主板自动开始升级。
8. 升级完成后，取出MicroSD卡，主板自动重启，完成整个更新固件的流程。
    ![](../../../rk3399_img/sdfirmwaretool_done.zh_CN.jpg)



## 准备需要的文件
因为格式化会丢失文件，现要重新准备MicroSD卡升级所需文件

* rksdfw.tag
* sd_boot_config.config
* sdupdate.img

```
mkdir my_tmp/
cd my_tmp/

#制作空的 rksdfw.tag
touch rksdfw.tag

#制作升级的配置文件 sd_boot_config.config
echo "#rockchip sdcard boot config file for factory\
loader_update = 0\
display_led = 1\
display_lcd = 1\
pcba_test = 0\
fw_update = 1\
demo_copy = 0" > sd_boot_config.config

# 制作待升级的固件 sdupdate.img
# 以固件 FIREFLY-RK3399-UBUNTU-18.04_DESKTOP-GPT.img 为例
cp ~/Downloads/FIREFLY-RK3399-UBUNTU-18.04_DESKTOP-GPT.img sdupdate.img

# 检查存在以下文件
ls rksdfw.tag  sd_boot_config.config  sdupdate.img
```


准备好 Firefly 提供的格式化脚本`flash.sh`
```
#!/bin/bash

DISK=$1
MNT=$2

sudo umount $DISK
sudo mkfs.ext4 -F $DISK
sudo mount $DISK $MNT
sudo cp rksdfw.tag  sd_boot_config.config  sdupdate.img $MNT
sudo umount $MNT
```

## 升级分区制作

<font color = "red">注意事项（必看）：</font>
* `[sdc]1` <font color = "red">请务必确认好进入的SD卡是哪一个设备节点</font>（是sdb或sdc还是其它），以免破坏本机系统或造成更大的生产损失。
* 如果确认是sdc这个盘符，那么操作的设备应该是`/dev/sdc1`而非其它。

```
# 检查存在以下文件
ls rksdfw.tag  sd_boot_config.config  sdupdate.img

# 给flash.sh执行权限
chmod +x flash.sh

sudo ./flash.sh /dev/[sdc]1 /mnt/
```