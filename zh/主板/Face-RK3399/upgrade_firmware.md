# 升级固件

## 前言

本文介绍了如何将主机上的固件文件
* 1.通过USB数据线，烧录到开发板的闪存中。升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。
* 2.生成U盘升级的固件，并用u盘对板子进行升级的方法。


## 1.USB 数据线升级的方法
### 准备工作

* Face-RK3399 开发板
* 固件
* 主机
* 双端公头USB数据线
* 20pin集成转接口一根

固件文件一般有两种：

* 单个统一固件 update.img, 将启动加载器、参数和所有分区镜像都打包到一起，用于固件发布。
* 多个分区镜像,如 kernel.img, rootfs.img, recovery.img 等，在开发阶段生成。
* 可以在这里找到已编译好的统一[[Face-RK3399固件]](http://www.t-firefly.com/doc/download/page/id/72.html#other_258)，下载后解压。也可以参考编译固件的说明自行编译。

主机操作系统支持：
> * Windows XP （32/64位）
> * Windows 7 (32/64位)
> * Windows 8 (32/64位)
> * Linux (32/64位)

### Windows

* 工具: [AndroidTool_Release_v2.63](http://www.t-firefly.com/doc/download/page/id/72.html#windows_256)

#### 安装 RK USB 驱动

下载 [ Release_DriverAssistant.zip](http://www.t-firefly.com/doc/download/page/id/72.html#other_255) ，解压，然后运行里面的 DriverInstall.exe 。   
为了所有设备都使用更新的驱动，请先选择"驱动卸载"，然后再选择"驱动安装"。   

![](../../../rk3399_img/Face-RK3399/upgrade_firmware1.png)

#### 连接设备

按照以下方式可以使设备进入升级模式

* 先断开电源适配器连接：
   * 双端公头USB数据线一端连接主机，一端连接开发板
   * 按住设备上的 RECOVERY （恢复）键并保持。
   * 接上电源
   * 大约两秒钟后，松开 RECOVERY 键。

主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备"Rockusb Device" 出现，如下图。如果没有，则需要返回上一步重新安装驱动。   

![](../../../rk3399_img/Face-RK3399/upgrade_firmware2.png)

### 烧写固件

下载 [AndroidTool](http://www.t-firefly.com/doc/download/page/id/72.html#windows_256)，解压，运行 AndroidTool_Release_v2.63 目录里面的 AndroidTool.exe（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图:
![](../../../rk3399_img/Face-RK3399/upgrade_firmware3.png)

#### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至"升级固件"页。
2. 按"固件"按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按"升级"按钮开始升级。
4. <font color=#ff0000 >如果升级失败，可以尝试先按"擦除Flash"按钮来擦除 Flash，然后再升级。</font>

**注意：如果你烧写的固件laoder版本与原来的机器的不一致，请在升级固件前先执行"擦除Flash"。** 

![](../../../rk3399_img/Face-RK3399/upgrade_firmware4.png)

#### 烧写分区映像

每个固件的分区可能不相同,请注意以下两点:
1. 使用`Androidtool_2.38`烧写`ubuntu(MBR)`和`Android7.1`固件时使用默认配置即可;
2. 使用`Androidtool_2.58`烧写`ubuntu(GPT)`使用默认配置即可，烧写`Android8.1`固件请先执行以下操作:
   
   <font color=#ff0000 >切换至"下载镜像页面"; 右键点击表格，选择"导入配置"; 选择rk3399-Android81.cfg</font>

烧写分区映像的步骤如下：

1. 切换至"下载镜像"页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击"执行"按钮开始升级，升级结束后设备会自动重启。

![](../../../rk3399_img/Face-RK3399/upgrade_firmware3.png)

### Linux

 Linux 下无须安装设备驱动，参照 Windows 章节连接设备则可。

* 工具:[Linux_Upgrade_Tool_1.34](http://www.t-firefly.com/doc/download/page/id/72.html#linux_256)

#### upgrade_tool

下载 [Linux_Upgrade_Tool_1.34](http://www.t-firefly.com/doc/download/page/id/72.html#linux_256), 并按以下方法安装到系统中，方便调用：   
```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

烧写统一固件 update.img：   
```
sudo upgrade_tool uf update.img
```

```
# 擦除flash 使用ef参数需要指定loader文件或者对应的update.img
sudo upgrade_tool ef update.img #update.img :你需要烧写的ubuntu固件
# 重新烧写
sudo upgrade_tool uf update.img 
```

烧写分区镜像：   
Ubuntu(MBR)、Android7.1、Android8.1,使用以下方式:
```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di resource /path/to/resource.img
sudo upgrade_tool di -p paramater   #烧写 parameter
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
```

Ubuntu(GPT),使用以下方式
```
sudo upgrade_tool ul $LOADER
sudo upgrade_tool di -p $PARAMETER
sudo upgrade_tool di -uboot $UBOOT
sudo upgrade_tool di -trust $TRUST
sudo upgrade_tool di -b $BOOT
sudo upgrade_tool di -r $RECOVERY
sudo upgrade_tool di -m $MISC
sudo upgrade_tool di -oem $OEM
sudo upgrade_tool di -userdata $USERDATA
sudo upgrade_tool di -rootfs $ROOTFS
```


如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 nand flash：   
```
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```
### 常见问题

#### 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《如何进入 MaskRom 模式》](maskrom_mode.md)


## 2.U盘OTA升级固件的方法

### 准备工作

* Face-RK3399 开发板
* OTA固件
* FAT32格式的u盘

### 生成OTA升级固件的方法

* 在下载的sdk源码编译完uboot 内核之后，在根目录下执行如下命令
```
make -j8 && ./mkimage.sh ota && make otapackage -j8 && ./FFTools/mkupdate/mkupdate.sh
```
执行完之后会在out/target/product/rk3399_firefly_face目录下生成xxx.zip文件，这个就是ota升级固件

### U盘OTA升级步骤
1.将上述xxx.zip重命名成update.zip

2.将update.zip 拷贝到FAT格式U盘的根目录下

3.在机器开机之后，将U盘插进 非OTG口的U盘接口中

4.等待若干秒，屏幕会弹出指示框，并点击Install按钮

5.等待机器重启即可，整个过程需要几分钟时间


### 注意事项
1.需要FAT32格式的U盘，其他格式U盘格式不支持

2.升级的U盘口需要使用非OTG口的另外的U盘接口

3.确保板子上旧固件也是./mkimage ota方式生成的固件

4.ota固件受时间戳限制，只能用时间戳新的固件升级旧时间戳的固件

5.升级完成之后，界面会弹出是否删除U盘OTA固件的提示，需慎点

6.由于是ota的打包固件方式，若客户想替换kernel.img和resource.img，单独烧录是不可行的，需要先执行./mkimage.sh ota 将kernel.img和resource.img打包进去boot.img，重新烧录boot.img

7.步骤4一般等待最多不超过1分钟，会弹出升级指示框。若出现超过一分钟不弹出显示框，需要按以下步骤：

	a.进入资源管理器,进入u盘,将u盘内的update.zip手动拷贝到sdcard(Internal Memory)根目录下。

	b.拷贝完成,断电重启机器。开机之后，等待若干秒，就会弹出升级指示框,可点击继续进行之后的升级步骤。

8.机器升级过程中如果出现如下图所示解密失败的报错，请点击"恢复平板电脑出厂设置"选项按钮，等待重启完成即可。

![](../../../rk3399_img/Face-RK3399/upgrade_firmware5.jpg)


