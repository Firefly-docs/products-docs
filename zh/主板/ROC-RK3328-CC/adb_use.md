# Adb 介绍

`Adb` 是 Android Debug Bridge 的简称，是 Android 的命令行调试工具，可以完成多种功能，如跟踪系统日志、上传下载文件、安装应用等。

## 准备工作

使用 `adb`时，你需要：

1. 使用公对公 USB 线连接电脑和板子的 USB OTG 口:

   ![](../../../rk3328_img/ROC-RK3328-CC/hw_board_usbconn.png)

2. 在跑 Android 的开发板上，选择 `Settings` -> `USB`，然后勾选 `Connect to PC` 选项。
3. 基于你的系统安装 adb 驱动和命令。

### Adb 在 Windows 下的安装

1. 安装 [Rockusb 驱动]。
2. 下载 [adb.zip](http://adbshell.com/upload/adb.zip)，然后解压到 `C:\adb`。

打开 `cmd` 窗口然后运行:

``` shell
C:\adb\adb shell
```

若成功就会进入 adb shell 。

### Adb 在 Ubuntu 下的安装

1. 安装 adb 工具:

    ``` shell
    sudo apt-get install android-tools-adb
    ```

2. 添加设备 ID:

    ``` shell
    mkdir -p ~/.android
    vi ~/.android/adb_usb.ini
    # add the following line:
    0x2207
    ```

3. 为非 root 用户添加 udev 规则：

    ``` shell
    sudo vi /etc/udev/rules.d/51-android.rules
    # add the following line：
    SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
    ```

4. 重载 udev 规则:

    ``` shell
    sudo udevadm control --reload-rules
    sudo udevadm trigger
    ```

5. 普通用户下重启 adb:

    ``` shell
    sudo adb kill-server
    adb start-server
    ```

然后就可以直接使用 adb 了, 如:

``` shell
adb shell
```

## 常用 Adb 命令

### 连接管理

列出所有连接设备以及它们的序列号：

``` shell
adb devices
```

若没有多连接设备，就必须用序列号来区分：

``` shell
export ANDROID_SERIAL=<device serial number>
adb shell ls
```

也可以用 TCP/IP 网络连接 Adb ：

``` shell
adb tcpip 5555
```

Adb 会在设备上重启并监听 5555 TCP 端口， 这个时候就可以拔出 USB 线了。

如果设备的 IP 地址为 192.168.1.100，执行以下命令连接:

``` shell
adb connect 192.168.1.100:5555
```

一旦连接，就可以执行 adb 命令了：

``` shell
adb shell ps
adb logcat
```

直到断开 adb 连接：

``` shell
adb disconnect 192.168.1.100:5555
```

### 调试

#### 查询系统日志

用法:

``` shell
adb logcat [option] [Application label]
```

示例:

``` shell
# 查看所有日志
adb logcat

# 仅查看部分日志
adb logcat -s WifiStateMachine StateMachine
```

#### 收集 Bug 报告

`adb bugreport` 用来收集错误报告和一些系统信息。

``` shell
adb bugreport

# 保存到本地，易于编辑和查看
adb bugreport >bugreport.txt
```

### 运行 shell

打开一个交互的 shell:

``` shell
adb shell
```

执行 shell 命令:

``` shell
adb shell ps
```

### Apk 管理

#### 安装 Apk

```text
adb install [option] example.apk

选项:
-l 转发锁定
-r 重新安装应用程序以保留原始数据
-s 安装到SD卡而不是内部存储
```

示例:

``` shell
# 安装 facebook.apk
adb install facebook.apk

# 升级 twitter.apk
adb install -r twitter.apk
```

若安装失败，检查下常见原因:

- `INSTALL_FAILED_ALREADY_EXISTS`: 尝试添加 `-r` 参数再次安装。
- `INSTALL_FAILED_SIGNATURE_ERROR`: APK 签名不一致，这可能是由于签名和调试版本的不同导致的。如果确认APK文件签名是正常的，可以使用 `adb uninstall` 命令卸载旧的应用程序，然后重新安装。
- `INSTALL_FAILED_INSUFFICIENT_STORAGE`: 存储空间不够。

#### 卸载 Apk

``` shell
adb uninstall apk_name
```

示例:

``` shell
adb uninstall com.android.chrome
```

apk 包的名称可以用下面的命令列出:

``` shell
adb shell pm list packages -f
...
package:/system/app/Bluetooth.apk=com.android.bluetooth
...
```

Apk文件路径和软件包名称用 `=` 分隔。

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
[下载页面]: http://www.t-firefly.com/doc/download/page/id/34.html
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