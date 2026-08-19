# 烧写 eMMC

## 启动模式

eMMC 一般都是直接焊在主板上，有些虽然是可插拔的，但没有专用的读卡器，因此需要使用板载烧写的方式来更新固件，即板上跑一个小系统，负责从主机或其它存储介质读取固件，再烧写到 eMMC 上。

取决于 eMMC 现存的内容，开发板有两种特殊的启动模式： [Rockusb 模式] 和 [Maskrom 模式]。

通常只需要进入 [Rockusb 模式] 即可升级现有的 Android 或 Ubuntu 系统。这种方式升级方式一般适用于 [RK 固件]或[分区映像]。

[Maskrom 模式] 则是系统未能识别到合法的启动设备而进入的模式。烧写[原始固件]到 eMMC 必须要进入该模式。

<a id="rockusb-mode"></a>

### Rockusb 模式

开发板启动后，CPU 如果在 eMMC 中找到有效的 IDB (IDentity Block)，它将从 eMMC 读取并加载 bootloader，并将执行控制权交给它。

如果 bootloader 检测到 Recovery 按钮按下并且 USB 已连接，它就会进入 [Rockusb 模式]，等待来自主机的命令。

进入 [Rockusb 模式] 的准备工作：

- 5V2A 电源适配器。
- 连接电源适配器和开发板的 Micro USB 线。
- 用来连接电脑 PC 以及开发板的公对公 USB 线。
- eMMC 。

操作步骤：

1. 将所有 USB 线（包括 Micro USB 线和公对公 USB 线）拔出开发板，以保持开发板断电。
2. 安装好 eMMC，拔出 SD 卡。
3. 使用公对公 USB 线将主机的 USB 端口与开发板的双层 USB 端口中靠近电路板的 OTG 端口相连：

   ![](../../../rk3328_img/ROC-RK3328-CC/hw_board_usbconn.png)

4. 按住开发板上的 RECOVERY 按键。
5. 将 Micro USB 线插入到开发板中，让开发板上电。
6. 等待大概 3 秒左右松开 RECOVERY 按键。

<a id="maskrom-mode"></a>

### Maskrom 模式

如果开发板上电后遇到以下情况之一：

- eMMC 内容为空。
- eMMC 上的 bootloader 损坏。
- 将 eMMC 数据/时钟引脚接地，eMMC读取数据失败。

CPU 在 eMMC 中就会找不到有效的 IDB (IDentity Block)，转而执行一段小型的 ROM 代码，等待主机通过 USB 上传 bootloader 来初始化 DDR 内存并进入升级状态。这种模式称为 [Maskrom 模式] 。

强制进入 [MaskRom 模式] 涉及到硬件操作，具有一定的风险，因此操作上需要 **非常谨慎** 。

进入 [Maskrom 模式] 的准备工作：

- 5V2A 电源适配器。
- 用来连接电源适配器和开发板的 Micro USB 线。
- 连接电脑 PC 和开发板的公对公 USB 线。
- 用于将 eMMC 时钟引脚短接到地的金属镊子。
- eMMC 。

操作步骤：

1. 将所有 USB 线（包括 Micro USB 线和公对公 USB 线）拔出开发板，以保持开发板断电。
2. 安装好 eMMC，拔出 SD 卡。
3. 使用公对公 USB 线将主机的 USB 端口与开发板的双层 USB 端口中靠近电路板的 OTG 端口相连：

    ![](../../../rk3328_img/ROC-RK3328-CC/hw_board_usbconn.png)

4. 找到开发板上预留的 eMMC 的 CLK 引脚和 GND 脚，见下图：

    ![](../../../rk3328_img/ROC-RK3328-CC/rk3328_maskrom_pads.jpg)

5. 用金属镊子短接 eMMC 的 CLK 和 GND 焊盘，并保持短接良好。
6. 将 Micro USB 线插入到开发板中，让开发板上电。
7. 保持住一秒后松开镊子。

## 下载固件

- [固件下载页面](https://community.t-firefly.com/doc/download/34)
  
**固件说明:固件分为 [原始固件] 与 [RK 固件] ，已经分类到不同的文件夹中，固件命名的日期最新则为最新的固件，其稳定性更好，请根据你所需要的固件类型选择正确的烧写工具。**

**Linux(GPT)固件：官方云盘提供的 Linux 固件中，其中包括 Ubuntu，Buildroot，Debian 等 Linux 系统，带 `GPT` 字样为 GPT 分区格式固件，统称为 Linux(GPT) 固件，按照 [编译 Linux固件(GPT)](linux_compile) 编译出来的固件也为 Linux(GPT) 固件。**

## 烧写工具

请根据所用主机的操作系统选择相应的烧写 eMMC 工具：

- 烧写 eMMC
    + 图形界面烧写工具：
        * [AndroidTool] (Windows)
    + 命令行烧写工具：
        * [upgrade_tool] (Linux)
        * [rkdeveloptool] (Linux)

<font color=#ff0000 size=3>提示</font>：[AndroidTool] 和 [upgrade_tool] 均支持烧写 [原始固件] 与 [RK 固件]，而 [rkdeveloptool] 仅支持烧写 [原始固件]。

<a id="upgrade-table"></a>

## 烧写须知

不同固件之间使用的烧写工具和烧写方法不一样的，请按照下面的表格进行烧写。

![](../../../rk3328_img/ROC-RK3328-CC/upgrade-table1.png)
![](../../../rk3328_img/ROC-RK3328-CC/upgrade-table2.png)

## AndroidTool

[AndroidTool] 是 Windows 下用来烧写[原始固件]、[RK 固件]和[分区映像]到 eMMC 的工具。

使用 [AndroidTool] 之前， 应先安装 [Rockusb 驱动]，然后再安装运行 [AndroidTool]。

<a id="rockusb-driver"></a>

### 安装 Rockusb 驱动

下载 [DriverAssistant](https://pan.baidu.com/s/1migPY1U#list/path=%2FPublic%2FDevBoard%2FROC-RK3328-CC%2FTools%2FRKTools%2Fwindows&parentPath=%2FPublic%2FDevBoard%2FROC-RK3328-CC
)， 解压后运行里面的 `DriverInstall.exe`：

![](../../../rk3328_img/ROC-RK3328-CC/started_driverassistant.png)

点击 "驱动安装" 按钮安装驱动；如果想卸载驱动，则点击 "驱动卸载" 按钮。

若设备处于 [Rockusb 模式] 或 [Maskrom 模式]，在设备管理器中会出现 "Rockusb Device"：

![](../../../rk3328_img/ROC-RK3328-CC/started_driverassistant_dev.png)

这表示驱动安装成功。

### 安装 AndroidTool

烧写工具选择与下载：

- 烧写原始固件：[AndroidTool_v2.39](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool.7z)
- 烧写 RK Android7.1 固件：[AndroidTool_v2.38](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_v2.38.rar)
- 烧写 RK Android8.1 固件：[AndroidTool_v2.58](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_V2.58.zip)
- 烧写 RK Linux(GPT) 固件：[AndroidTool_v2.58](http://download.t-firefly.com/product/RK3328/Tools/AndroidTool/AndroidTool_Release_V2.58.zip)

下载并解压后运行里面的 `AndroidTool.exe`：

![](../../../rk3328_img/ROC-RK3328-CC/androidtool.zh_CN.png)

若设备处于 [Rockusb 模式]，状态行将显示 "发现一个LOADER设备"。

若设备处于 [Maskrom 模式]，状态行将显示 "发现一个MASKROM设备"。

### 烧写原始固件

[原始固件]需要从 eMMC 的偏移地址为 0 的位置开始烧写。但在 [Rockusb 模式] 下无法做到这点，因为所有 LBA 写入操作会偏移 0x2000 个扇区（即 LBA0 对应于存储设备上第 0x2000 个扇区）。因此，开发板必须强制进入 [Maskrom 模式] 才能烧写[原始固件]。

使用 `AndroidTool_v2.39` 烧写[原始固件]到 eMMC 的步骤如下：

1. 强制设备进入 [Maskrom 模式]。
2. 运行 [AndroidTool]。
3. 打开 “下载镜像” 制表页。
4. 保持表格的第一行不变， 使用默认的 “Loader” 文件。
5. 点击第二行右侧的空白单元格，在弹出的文件对话框里打开[原始固件]文件。
6. 点击 "执行" 按钮开始烧写。

![](../../../rk3328_img/ROC-RK3328-CC/androidtool_flash_image.zh_CN.png)

### 烧写 RK 固件

使用 `AndroidTool` 烧写 [RK 固件] 到 eMMC 的步骤如下：

1. 强制设备进入 [Rockusb 模式] 或 [Maskrom 模式]。
2. 运行 [AndroidTool]。
3. 打开 "升级固件" 制表页。
4. 点击 "固件" 按钮， 在弹出的文件对话框里打开[RK 固件]文件。
5. 固件版本号、loader 版本号和芯片信息会从固件中读取并显示。
6. 点击 "升级" 按钮烧录。

然后烧写不同的固件或者出现烧写失败，请按照 [烧写须知](flash_emmc.html#upgrade-table) 先擦除后升级。

### 烧写分区映像

取决开发板原有的固件，烧写 [分区映像] 到 eMMC 会有所不同。

**[原始固件]**

 使用 `AndroidTool_v2.39` 烧写 [分区映像] 到 eMMC 的步骤如下：

1. 强制设备进入 [Rockusb 模式] 或 [Maskrom 模式]。
2. 运行 `AndroidTool`。
3. 打开 "下载镜像" 制表页。
4. 保持表格第一行不变。
5. 鼠标右键点击其它行，在弹出菜单中选择 "删除项" ，重复直至删除第一行除外的所有行。

   ![](../../../rk3328_img/ROC-RK3328-CC/androidtool_del.zh_CN.png)

6. 鼠标右键点击表格，在弹出菜单中选择 "添加项" 以便添加[分区映像]：
    + 选中第一个单元格上的复选框。
    + 填入 `parameter.txt` 中该分区的起始扇区作为烧写地址（如果是 [Maskrom 模式] 则须再加上 `0x2000`）。
    + 单击右侧空白单元格，在弹出的文件对话框里打开对应的[分区映像]文件。

    ![](../../../rk3328_img/ROC-RK3328-CC/androidtool_add.zh_CN.png)

7. 点击 "执行" 按钮烧录。

**注意**：

- 您可以通过重复步骤 6 将多个分区映像烧写到闪存。
- 通过取消选中地址单元格前面的复选框，可以跳过此分区的烧写。
- 在 [Maskrom 模式] 中， `parameter.txt` 中分区的起始扇区必须再加上 `0x2000` 作为烧写地址。参见[《分区偏移量》](#partition-offset)一章以便使用脚本获取该地址。


**RK 固件**

使用 `Androidtool_2.38` 烧写 `Android7.1` RK 固件时使用默认配置即可;

使用 `Androidtool_2.58` 烧写 `Android8.1` RK 固件使用 `rk3328-Android81.cfg` 配置，右击空白部分，点击`导入配置`，选择对应的配置文件。

使用 `Androidtool_2.58` 烧写 `Linux(GPT)分区` RK 固件使用默认配置即可。

使用 `AndroidTool` 烧写 [分区映像] 到 eMMC 的步骤如下：

1. 切换至”下载镜像”页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击”执行”按钮开始升级，升级结束后设备会自动重启。

![](../../../rk3328_img/ROC-RK3328-CC/android-v2.58-rk-linux.png)

<a id="upgrade-tool"></a>

## upgrade_tool

[upgrade_tool] 是 Linux 下用来烧写[原始固件]、[RK 固件]和[分区映像]到 eMMC 的工具，是 Rockchip 提供的闭源命令行工具。

### 安装 upgrade_tool

工具选择与下载：

- 烧写`原始固件`或者 `Android7.1` ：[upgrade_tool_v1.24](http://download.t-firefly.com/product/RK3328/Tools/Linux_Upgrade_Tool/Linux_Upgrade_Tool_v1.24.zip)
- 烧写 `Linux(GPT) RK` 固件或者 `Android8.1` ：[upgrade_tool_v1.34](http://download.t-firefly.com/product/RK3328/Tools/Linux_Upgrade_Tool/Linux_Upgrade_Tool_1.34.zip)

下载 [upgrade_tool]，并安装到 Linux 系统上：

    unzip Linux_Upgrade_Tool_v1.24.zip
    cd Linux_UpgradeTool_v1.24
    sudo mv upgrade_tool /usr/local/bin
    sudo chown root:root /usr/local/bin/upgrade_tool
    sudo chmod 0755 /usr/local/bin/upgrade_tool

然后根据[此处](#udev)的说明去添加 `udev` 规则。这是为了让普通用户有权限烧写 Rockchip 设备。如果跳过这步，那么所有的烧写命令均需在前面加 `sudo` 才能成功执行。

### 烧写原始固件

[原始固件]需要从 eMMC 的偏移地址为 0 的位置开始烧写。但在 [Rockusb 模式] 下所有 LBA 写入操作会偏移 0x2000 个扇区（即 LBA0 对应于存储设备上的第 0x2000 个扇区）。因此，开发板必须强制进入 [Maskrom 模式] 才能烧写[原始固件]。

使用 [upgrade_tool] 烧写[原始固件]到 eMMC 的步骤如下：

1. 强制设备进入 [Maskrom 模式]。
2. 运行以下命令：
    ``` shell
    upgrade_tool db     out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    upgrade_tool wl 0x0 out/system.img
    upgrade_tool rd     # 重置并启动设备
    ```

其中：

- `rk3328_loader_ddr786_v1.06.243.bin` 是编译 `U-Boot` 时复制进去的 loader 文件，也直接到[此处](https://t-firefly.oss-cn-hangzhou.aliyuncs.com/product/RK3328/Firmware/rk3328_loader_ddr786_v1.06.243.bin)下载 `rk3328_loader_xxx.bin` 文件。
- `system.img` 是打包后的[原始固件]，也可以是官网上下载并解压后的[原始固件]文件。

### 烧写 RK 固件

使用 [upgrade_tool] 烧写 [RK 固件]到 eMMC 的步骤如下：

1. 原固件与需要烧写的固件为同一固件只需进入[Rockusb 模式]，不同固件烧写需要进入 [Maskrom 模式]。
2. 使用 `upgrade_tool` 烧写 RK 固件：
``` 
upgrade_tool uf path/update.img
```

然后烧写不同的固件或者出现烧写失败，请按照 [烧写须知](flash_emmc.html#upgrade-table) 先擦除后升级。
```
upgrade_tool ef path/update.img  # 擦除闪存
```

### 烧写分区映像

取决开发板原有的固件，烧写[分区映像]到 eMMC 的指令会有所不同。

**原始固件**

如果开发板原有系统是[原始固件]，那么很可能使用了 `GPT` 分区方案。每个分区的预定义偏移量和大小可以在 SDK 里的 `build/partitions.sh` 中找到，可以参考[《分区偏移量》](#partition-offset)一章。

使用 [upgrade_tool] 烧写 [分区映像]的步骤如下：

1. 强制设备进入 [Maskrom 模式]
2. 使用 [upgrade_tool] 烧写分区映像：
    ``` shell
    upgrade_tool db         out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    upgrade_tool wl 0x40    out/u-boot/idbloader.img
    upgrade_tool wl 0x4000  out/u-boot/uboot.img
    upgrade_tool wl 0x6000  out/u-boot/trust.img
    upgrade_tool wl 0x8000  out/boot.img
    upgrade_tool wl 0x40000 out/linaro-rootfs.img
    upgrade_tool rd         # 重置并启动设备
    ```

**RK 固件**

如果开发板原有系统是 [RK 固件]，那么它使用 `parameter` 文件作为分区方案，这样就可直接使用分区名称来烧写分区映像：

1. 强制设备进入 [Rockusb 模式]。
2. 使用 [upgrade_tool] 烧写分区映像：

Android7.1 使用以下方式： 
```
upgrade_tool di -b boot.img
upgrade_tool di -k kernel.img
upgrade_tool di -s system.img
upgrade_tool di -r recovery.img
upgrade_tool di -m misc.img
upgrade_tool di resource resource.img
upgrade_tool di -p parameter   # 烧写 parameter
upgrade_tool ul bootloader.bin # 烧写 bootloader
```

Android8.1 使用以下方式：
```
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
sudo upgrade_tool di -p paramater   # 烧写 parameter
sudo upgrade_tool di -uboot uboot.img
sudo upgrade_tool di -trust trust.img
sudo upgrade_tool di -m misc.img
sudo upgrade_tool di -baseparameter baseparameter.img
sudo upgrade_tool di -b boot.img
sudo upgrade_tool di -k kernel.img
sudo upgrade_tool di -resource resource.img
sudo upgrade_tool di -r recovery.img
sudo upgrade_tool di -s system.img
sudo upgrade_tool di -vendor vendor.img
sudo upgrade_tool di -oem oem.img
```

Linux(GPT) 使用以下方式：
```
upgrade_tool ul $LOADER
upgrade_tool di -p $PARAMETER
upgrade_tool di -uboot $UBOOT
upgrade_tool di -trust $TRUST
upgrade_tool di -b $BOOT
upgrade_tool di -r $RECOVERY
upgrade_tool di -m $MISC
upgrade_tool di -oem $OEM
upgrade_tool di -userdata $USERDATA
upgrade_tool di -rootfs $ROOTFS
```

注意：

- `-b` 是 `boot` 分区的预定义缩写。如果没有缩写可用，请改为分区名称，例如上述例子中的 `resource`。
- 可根据 [《参数文件格式》](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf) 这份文档的说明自定义内核参数和分区布局。分区布局更改后，必须先重新烧写该 `parameter` 文件，方可重新烧写受影响的相应分区。

### 常见问题

如果由于闪存问题而出现错误，可以尝试使用低格或擦除闪存：

    upgrade_tool lf   # 低格闪存
    upgrade_tool ef   # 擦除闪存

## rkdeveloptool

[rkdeveloptool] 是 Linux 下用来烧写[原始固件]和[分区映像]到 eMMC 的工具。它**不支持**烧写 [RK 固件]。

[rkdeveloptool] 是由 Rockchip 开发的命令行烧写工具，是闭源工具 [upgrade_tool] 的开源替代品。在日常使用中，一般都能取代 [upgrade_tool]。

### 安装 rkdeveloptool

首先是下载、编译和安装 [rkdeveloptool]：

    #install libusb and libudev
    sudo apt-get install pkg-config libusb-1.0 libudev-dev libusb-1.0-0-dev dh-autoreconf
    # clone source and make
    git clone https://github.com/rockchip-linux/rkdeveloptool
    cd rkdeveloptool
    autoreconf -i
    ./configure
    make
    sudo make install

然后根据[此处](#udev)的说明去添加 `udev` 规则。这是为了让普通用户有权限烧写 Rockchip 设备。如果跳过这步，则所有的烧写命令均需在前面加 `sudo` 才能成功执行。

### 烧写原始固件

[原始固件]需要从 eMMC 的偏移地址为 0 的位置开始烧写。但在 [Rockusb 模式] 下所有 LBA 写入操作会偏移 0x2000 个扇区（即 LBA0 对应于存储设备上的第 0x2000 个扇区）。因此，开发板必须强制进入 [Maskrom 模式] 才能烧写[原始固件]。

使用 [rkdeveloptool] 烧写[原始固件]到 eMMC 的步骤如下：

1. 强制设备进入 [Maskrom 模式]。
2. 运行以下命令：
    ``` shell
    rkdeveloptool db           out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    rkdeveloptool wl 0x0       out/system.img
    rkdeveloptool rd           # 重置并启动设备
    ```

其中：

- `rk3328_loader_ddr786_v1.06.243.bin` 是编译 `U-boot` 时复制进去的 loader 文件，也直接到[此处](https://github.com/rockchip-linux/rkbin/tree/master/rk33)下载 `rk3328_loader_xxx.bin` 文件。
- `system.img` 是打包后的[原始固件]，也可以是官网上下载并解压后的[原始固件]文件。

### 烧写分区映像

以下的说明**仅适用**于开发板原有系统是[原始固件]时的[分区映像]烧写。每个分区的预定义偏移量和大小可以在 SDK 里的 `build/partitions.sh` 中找到，可以参考[《分区偏移量》](#partition-offset)一章。

使用 [rkdeveloptool] 烧写[分区映像]的步骤如下：

1. 强制设备进入 [Maskrom 模式]。
2. 运行以下命令：
    ``` shell
    rkdeveloptool db           out/u-boot/rk3328_loader_ddr786_v1.06.243.bin
    rkdeveloptool wl 0x40      out/u-boot/idbloader.img
    rkdeveloptool wl 0x4000    out/u-boot/uboot.img
    rkdeveloptool wl 0x6000    out/u-boot/trust.img
    rkdeveloptool wl 0x8000    out/boot.img
    rkdeveloptool wl 0x40000   out/linaro-rootfs.img
    rkdeveloptool rd           # 重置并启动设备
    ```

## udev

创建 `/etc/udev/rules.d/99-rk-rockusb.rules`，并插入以下内容[1](https://github.com/rockchip-linux/rkdeveloptool/blob/master/99-rk-rockusb.rules)。 如有必要，用实际 Linux 组替换 `users` 组：

``` shell
SUBSYSTEM!="usb", GOTO="end_rules"

# RK3036
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="301a", MODE="0666", GROUP="users"
# RK3229
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320b", MODE="0666", GROUP="users"
# RK3288
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320a", MODE="0666", GROUP="users"
# RK3328
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="320c", MODE="0666", GROUP="users"
# RK3368
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="330a", MODE="0666", GROUP="users"
# RK3399
ATTRS{idVendor}=="2207", ATTRS{idProduct}=="330c", MODE="0666", GROUP="users"

LABEL="end_rules"
```

重新加载 udev 规则：

    sudo udevadm control --reload-rules
    sudo udevadm trigger

<a id="partition-offset"></a>

## 分区偏移量

### GPT 分区

如果使用[原始固件]，那么系统很可能使用了 `GPT` 分区方案。每个分区的预定义偏移量和大小可以在 SDK 里的 `build/partitions.sh` 中找到。

分区映像的偏移量可以通过以下命令获得（假设位于 Firefly Linux SDK 的目录中）：

    (. build/partitions.sh ; set | grep _START | \
    while read line; do start=${line%=*}; \
    printf "%-10s 0x%08x\n" ${start%_START*} ${!start}; done )

会得到：

    ATF        0x00006000
    BOOT       0x00008000
    LOADER1    0x00000040
    LOADER2    0x00004000
    RESERVED1  0x00001f80
    RESERVED2  0x00002000
    ROOTFS     0x00040000
    SYSTEM     0x00000000

### parameter

如果使用 [RK 固件]，那么系统是使用 `parameter.txt` 文件来定义分区方案，该文件的格式参见文档[《参数文件格式》](http://www.t-firefly.com/download/Firefly-RK3399/docs/Rockchip%20Parameter%20File%20Format%20Ver1.3.pdf) 。

这里有一个 Linux 下的 Bash 脚本能列出 `parameter.txt` 中的分区偏移量：

``` shell
#!/bin/sh

PARAMETER_FILE="$1"
[ -f "$PARAMETER_FILE" ] || { echo "Usage: $0 <parameter_file>"; exit 1; }

show_table() {
    echo "$1"
    echo "--------"
    printf "%-20s %-10s %s\n" "NAME" "OFFSET" "LENGTH"
    for PARTITION in `cat ${PARAMETER_FILE} | grep '^CMDLINE' | sed 's/ //g' | sed 's/.*:\(0x.*[^)])\).*/\1/' | sed 's/,/ /g'`; do
        NAME=`echo ${PARTITION} | sed 's/\(.*\)(\(.*\))/\2/'`
        START=`echo ${PARTITION} | sed 's/.*@\(.*\)(.*)/\1/'`
        LENGTH=`echo ${PARTITION} | sed 's/\(.*\)@.*/\1/'`
        START=$((START + $2))
        printf "%-20s 0x%08x %s\n" $NAME $START $LENGTH
    done
}

show_table "Rockusb Mode" 0
echo
show_table "Maskrom Mode" 0x2000
```

将这个脚本保存为 `/usr/local/bin/show_rk_parameter.sh` ，然后添加脚本的执行权限。

下面是一个显示 `RK3328 Android SDK` 中定义的分区偏移量的例子：

```text
$ show_rk_parameter.sh device/rockchip/rk3328/parameter.txt
Rockusb Mode
--------
NAME                 OFFSET     LENGTH
uboot                0x00002000 0x00002000
trust                0x00004000 0x00004000
misc                 0x00008000 0x00002000
baseparamer          0x0000a000 0x00000800
resource             0x0000a800 0x00007800
kernel               0x00012000 0x00010000
boot                 0x00022000 0x00010000
recovery             0x00032000 0x00010000
backup               0x00042000 0x00020000
cache                0x00062000 0x00040000
metadata             0x000a2000 0x00008000
kpanic               0x000aa000 0x00002000
system               0x000ac000 0x00300000
userdata             0x003ac000 -

Maskrom Mode
--------
NAME                 OFFSET     LENGTH
uboot                0x00004000 0x00002000
trust                0x00006000 0x00004000
misc                 0x0000a000 0x00002000
baseparamer          0x0000c000 0x00000800
resource             0x0000c800 0x00007800
kernel               0x00014000 0x00010000
boot                 0x00024000 0x00010000
recovery             0x00034000 0x00010000
backup               0x00044000 0x00020000
cache                0x00064000 0x00040000
metadata             0x000a4000 0x00008000
kpanic               0x000ac000 0x00002000
system               0x000ae000 0x00300000
userdata             0x003ae000 -
```

[《上手指南》]: getting_started.md
[《常见问题解答》]: faqs.md
[《串口调试》]: debug.md
[《编译 Linux 根文件系统》]: linux_build_ubuntu_rootfs.md
[联系方式]: resource.md#社区
[原始固件]: getting_started.md#raw-firmware-format
[RK 固件]: getting_started.md#rk-firmware-format
[分区映像]: getting_started.md#partition-image
[SDCard Installer]: flash_sd.md#sdcard-installer
[Etcher]: flash_sd.md#etcher
[dd]: flash_sd.md#dd
[SD Firmware Tool]: flash_sd.md#sd-firmware-tool
[AndroidTool]: flash_emmc.md#androidtool
[upgrade_tool]: flash_emmc.md#upgrade-tool
[rkdeveloptool]: flash_emmc.md#rkdeveloptool
[Rockusb 模式]: flash_emmc.md#rockusb-mode
[Maskrom 模式]: flash_emmc.md#maskrom-mode
[Rockusb 驱动]: flash_emmc.md#rockusb-driver
[ROC-RK3328-CC]: http://www.t-firefly.com/product/rocrk3328cc.html "ROC-RK3328-CC 官网"
[下载页面]: https://community.t-firefly.com/doc/download/34
[论坛]: http://bbs.t-firefly.com
[脸书]: https://www.facebook.com/TeeFirefly
[Google+]: https://plus.google.com/u/0/communities/115232561394327947761
[油管]: https://www.youtube.com/channel/UCk7odZvUrTG0on8HXnBT7gA
[推特]: https://twitter.com/TeeFirefly
[在线商城]: http://store.t-firefly.com
[USB 转串口适配器]: https://store.t-firefly.com/goods.php?id=24
[5V2A 电源适配器]: https://store.t-firefly.com/goods.php?id=69
[eMMC 闪存]: https://store.t-firefly.com/goods.php?id=71
[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map