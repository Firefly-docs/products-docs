# 使用 USB 线升级固件

本文包含完整的 USB 固件升级流程，所有步骤都可以在本页完成，无需再打开其他指南。

## 1. 准备固件和烧写工具

### 准备设备

* RK182X开发套件 开发板
* [固件](https://community.t-firefly.com/doc/download/369)
* 主机电脑
* Type-A 数据线

固件可以通过编译 SDK 获得，也可以从[资源下载页面](https://community.t-firefly.com/doc/download/369)下载公版统一固件。

### 固件格式

固件文件一般有以下两种格式：

* **单个统一固件**：将分区表、bootloader、uboot、kernel、system 等分区文件打包成一个文件。烧写后会更新主板上的全部分区，并擦除主板原有数据。
* **多个分区镜像**：开发阶段生成的分区表、bootloader、kernel 等独立文件。烧写单个镜像只会更新对应分区，适合开发调试。

> 通过统一固件打包工具，可以将统一固件解包为多个分区镜像，也可以将多个分区镜像合并为统一固件。

### 安装烧写工具

#### Windows

1. 下载 [Release_DriverAssistant.zip](https://community.t-firefly.com/doc/download/369)，解压后运行 `DriverInstall.exe`。
2. 为确保所有设备使用更新后的驱动，请先选择“驱动卸载”，再选择“驱动安装”。

![](../../../gs1-n2_img/common/upgrade_firmware_install_rk_usb.jpg)

3. 可以单独下载 [AndroidTool](https://community.t-firefly.com/doc/download/369)，解压后运行 `RKDevTool_Release_v2.xx` 目录中的 `RKDevTool.exe`。如果使用 Windows 7/8，请以管理员身份运行。

为避免烧写工具版本导致烧写失败，建议优先使用公版固件压缩包中自带的工具：

```
ITX-3588J_Android12_HDMI_220308
├── ITX-3588J_Android12_HDMI_220308.img
├── linux
│   └── Linux_Upgrade_Tool_v1.65.zip
└── windows
    ├── DriverAssitant_v5.1.1.zip
    └── RKDevTool_Release_v2.84.zip
```

![](../../../gs1-n2_img/common/upgrade_firmware_androidtool_zh.png)

#### Linux

Linux 下无需安装设备驱动。

下载 [Linux_Upgrade_Tool](https://community.t-firefly.com/doc/download/369)，并安装到系统中：

```
unzip Linux_Upgrade_Tool_xxxx.zip
cd Linux_UpgradeTool_xxxx
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
sudo chmod a+x /usr/local/bin/upgrade_tool
```

下载 [Linux_adb_fastboot](https://community.t-firefly.com/doc/download/369)，并安装到系统中：

```
sudo mv adb /usr/local/bin
sudo chown root:root /usr/local/bin/adb
sudo chmod a+x /usr/local/bin/adb

sudo mv fastboot /usr/local/bin
sudo chown root:root /usr/local/bin/fastboot
sudo chmod a+x /usr/local/bin/fastboot
```

## 2. 进入并检查 MaskRom 模式

RK182X 开发套件不支持 Loader 模式。通过 USB 升级固件时，需要使用主板上的 `MaskRom` 按键。

### 进入 MaskRom 模式

1. 断开开发套件电源。
2. 将拨码开关 `USB SEL` 切换到 `1`。
3. 使用 Type-A 数据线连接 OTG 接口和主机电脑。
4. 按住主板上的 `MaskRom` 按键。
5. 保持按键按下，同时给开发套件上电。
6. 使用烧写工具检查是否发现 MaskRom 设备。
7. 设备被识别后，松开按键。

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/usb_sel.png)

![](../../../gs1-n2_img/AIO-GS1N2-RK182X/usb_otg.png)

![](../../../gs1-n2_img/common/upgrade_maskrom_zh.png)

### 检查 MaskRom 模式

#### Windows

AndroidTool 的设备列表中应显示 MaskRom 设备。如果没有识别，请检查 USB 驱动、Type-A 数据线、`USB SEL` 拨码开关和 OTG 连接。

#### Linux

运行 `upgrade_tool`，检查连接设备是否显示为 MaskRom：

```shell
sudo upgrade_tool
```

## 3. 烧写固件

### Windows

#### 烧写统一固件 `update.img`

1. 切换到 **Upgrade Firmware** 页面。
2. 点击 **Firmware**，打开待烧写的固件文件。
3. 点击 **Upgrade** 开始烧写。
4. 如果升级失败，可以先点击 **EraseFlash** 擦除 Flash，再重新升级。

![](../../../gs1-n2_img/common/upgrade_firmware_erase_flash_zh.png)

#### 烧写分区镜像

1. 切换到 **Download Image** 页面。
2. 勾选需要烧写的分区。
3. 确认每个镜像文件的路径正确。
4. 点击 **Run** 开始升级，升级结束后设备会自动重启。

![](../../../gs1-n2_img/common/upgrade_firmware_androidtool_zh.png)

### Linux

#### 烧写统一固件 `update.img`

```shell
sudo upgrade_tool uf update.img
```

如果升级失败，可以先擦除 Flash，再重新烧写：

```shell
# ef 参数需要指定 bootloader 文件或对应的 update.img。
sudo upgrade_tool ef update.img
sudo upgrade_tool uf update.img
```

#### 烧写分区镜像

```shell
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di -u /path/to/uboot.img
sudo upgrade_tool di -dtbo /path/to/dtbo.img
sudo upgrade_tool di -p parameter
sudo upgrade_tool ul bootloader.bin
```

如果因 Flash 问题导致升级失败，可以尝试低级格式化并擦除 eMMC：

```shell
sudo upgrade_tool lf update.img
sudo upgrade_tool ef update.img
```

#### Fastboot 烧写动态分区

```shell
adb reboot fastboot
sudo fastboot flash vendor vendor.img
sudo fastboot flash system system.img
sudo fastboot reboot
```

## 烧写失败排查

如果烧写过程中出现 `Download Boot Fail` 或其他错误，请先检查 USB 线缆和电脑 USB 接口。线材质量差或 USB 接口供电能力不足都可能导致烧写失败。

![](../../../gs1-n2_img/common/upgrade_firmware_download_fail.png)
