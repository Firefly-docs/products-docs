# FAQ

## Q：如何确认 NVMe SSD 是否正常？

a. 设备上电开机
b. 按 `ESC` 键进入 UEFI Menu（需连接 USB 键盘）
c. 选择 `Boot Manager` 进入启动管理器
d. 菜单中能看到 NVMe SSD 设备，则说明 NVMe SSD 工作正常

eg:

<center>

<img alt="" src="../../../nvidia_img/uefi_boot_manager.png" width="400">
</center>

## Q：升级固件前需要做哪些准备？

**A：** 请准备一台 Ubuntu 22.04 的 PC，PC 需要支持 NFS 服务，且升级过程中可能用到 `sshpass` 等命令，如系统缺失需自行安装；同时准备一根 Type-C 数据线，用于连接整机的 OTG 口与 PC。

固件包可从 Firefly [下载页面](https://community.t-firefly.com/doc/download/346)获取。

## Q：如何让整机进入 Recovery 模式？

**A：** 按以下步骤操作：

* EC-ThorT5000 先断电
* 使用 Type-C 数据线连接 EC-ThorT5000 的 OTG 口和 PC 端
* 长按 EC-ThorT5000 的 Recovery 键
* EC-ThorT5000 供电
* 释放 EC-ThorT5000 的 Recovery 键

## Q：如何确认整机已进入 Recovery 模式？

**A：** 在 PC 上执行 `lsusb` 命令，当看到 `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.` 字样时，即说明整机已进入 Recovery 模式。其中：

* `<bbb>` 为任意三位数
* `<ddd>` 为任意三位数
* `<nnnn>` 为四位数，与核心板模组型号对应：
    * `7026` ：Jetson T5000 (P3834-0008 with 128GB)

## Q：按步骤操作后，`lsusb` 中没有看到 Nvidia 设备怎么办？

**A：** 请依次排查：

* 确认操作顺序是否正确：先断电，连接 Type-C 线，按住 Recovery 键不放，再上电，最后释放 Recovery 键
* 更换 Type-C 数据线（部分线材仅支持充电，无法传输数据）或更换 PC 的 USB 端口
* 确认 Type-C 线连接的是整机的 **OTG** 口，而非 Debug 口

## Q：如何烧写固件？如何判断升级成功？

**A：** 下载固件包并解压后，进入固件包目录执行烧写命令：

```
# 解压固件包
tar xf flashImage.tar.gz
# 进入固件包
cd Linux_for_Tegra
# 升级固件
sudo ./l4t_initrd_flash.sh firefly-aio-thor-t5000 internal
```

如果一切顺利，升级成功后终端会出现 `Flash is successful` 等字样：

```
Flash is successful
Reboot device
Cleaning up...
```

完整日志保存在固件包的 `Linux_for_Tegra/initrdlog/` 目录下，详细的烧写步骤请参阅[升级固件](upgrade_firmware.md)。

## Q：烧写失败或中途出错怎么办？

**A：** 可以尝试以下方法：

* 重新按上述步骤进入 Recovery 模式后再次执行烧写命令
* 更换 Type-C 数据线或 PC 的 USB 端口，排除线材与接口接触不良问题
* 确认整机供电正常，升级过程中请勿给整机断电
* 多次失败时，查看固件包 `Linux_for_Tegra/initrdlog/` 下的日志定位问题

## Q：固件版本与 JetPack 有什么关系？

**A：** 固件包按 JetPack 版本发布（例如 R36.4 对应 JetPack 6.2，R38.4 对应 JetPack 7.1），不同版本固件包内的烧写脚本和命令可能不同。下载时请注意固件包对应的 JetPack/L4T 版本，烧写步骤以[升级固件](upgrade_firmware.md)页面中对应版本的说明为准。