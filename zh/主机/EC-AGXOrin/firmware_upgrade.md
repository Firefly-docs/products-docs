# 更新固件

## 升级前准备

PC 系统要求：Ubuntu22.04, 需要支持 NFS 服务，且在升级过程中可能会遇到一些命令缺失，如：`sshpass` 等命令，需用户自行安装。

## EC-AGXOrin 进入烧录模式
* EC-AGXOrin 先断电
* 使用 Type-C 数据线连接 EC-AGXOrin 的 OTG 口和 PC 端
* 长按 EC-AGXOrin 的 Recovery 键
* EC-AGXOrin 供电
* 释放 EC-AGXOrin 的 Recovery 键
* 检查 EC-AGXOrin 是否进入 Recovery 模式
    * 使用 `lsusb` 命令， 当你看到 `Bus <bbb> Device <ddd>: ID 0955: <nnnn> Nvidia Corp.` 时，即进入了 Recovery 模式。
        * `<bbb>` 任何三位数
        * `<ddd>` 任何三位数
        * `<nnnn>` 四位数
            * `7023` Jetson AGX Orin 64GB


## R36.4 (JetPack 6.2)
### 下载固件包

可直接在 Firefly [下载页面](https://www.t-firefly.com/doc/download/335.html)进行下载

下载后，执行 tar 解压：

```
tar xf mfi_firefly-aio-agx-orin.tar.gz
```

解压后，进入固件包内进行升级

```
# 进入固件包
cd mfi_firefly-aio-agx-orin
# 升级固件
sudo ./tools/kernel_flash/l4t_initrd_flash.sh --flash-only --massflash 1 --network usb0
```

如果一切顺利，一般升级成功后，会出现以下字段，表示升级成功：
```
Copied 16896 bytes from /mnt/internal/gpt_secondary_3_0.bin to address 0x03ffbe00 in flash
[ 233]: l4t_flash_from_kernel: Successfully flash the qspi
[ 233]: l4t_flash_from_kernel: Flashing success
[ 233]: l4t_flash_from_kernel: The device size indicated in the partition layout xml is smaller than the actual size. This utility will try to fix the GPT.
Flash is successful
Reboot device
Cleaning up...
Log is saved to Linux_for_Tegra/initrdlog/flash_1-2_0_20250527-153418.log
```
