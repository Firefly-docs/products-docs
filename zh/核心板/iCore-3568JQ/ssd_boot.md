## SSD 启动

本章节介绍如何把固件烧录到 SSD，并从 SSD 启动系统。

准备工作：
- RK356x 平台
- 良好的 M.2 SSD 或者 SATA SSD
- 良好的 Type-C 数据线
- 更新到最新的 Linux SDK，进入 u-boot 目录，确保有以下提交：
```bash
commit 9341129a705816604b7883313857307f248b57e5
Author: weicz <wcz@t-chip.com.cn>
Date:   Mon Mar 21 11:43:00 2022 +0800

    configs: add firefly-rk3568-usb/pcie/sata.config
    
    Signed-off-by: weicz <wcz@t-chip.com.cn>
```

### M.2 SSD 启动

1、进入 u-boot 目录，执行如下命令编译与打包引导固件：
```bash
./make.sh firefly-rk3568-pcie --spl-new
```

生成的固件有`rk356x_spl_loader_*.bin`与`uboot.img`。

2、RK356x 进入 Maskrom 模式，把引导固件烧录到 Nor Flash（默认）：

Windows 平台下选中`Loader`、`Parameter`与`Uboot`并烧录：

![](../../../rk356x_img/rkdevtool_to_nor.png)

Linux 平台下烧录`Loader`、`Parameter`与`Uboot`：
```bash
upgrade_tool ul rk356x_spl_loader_*.bin
upgrade_tool di -p parameter.txt
upgrade_tool di -u uboot.img
```

3、重启 RK356x，接上串口会看到如下打印：
```bash
Bootdev: nvme 0
```

这说明 RK356x 在 uboot 阶段可以识别到 M.2 SSD，此时因为 M.2 SSD 并没有固件，因此 uboot 会加载内核失败！

4、在 uboot shell 输入以下命令，可把系统固件烧录到 M.2 SSD：
```bash
=> rockusb 0 nvme 0
```

注意的是如果设备使用了 extboot，则需要进行取消，目前 M.2 SSD 在 extboot 下兼容性较差。

取消 extboot：

```diff
diff --git a/rk356x/firefly-rk3568-ubuntu.mk b/rk356x/firefly-rk3568-ubuntu.mk
index 8badc34..ee1610f 100644
--- a/rk356x/firefly-rk3568-ubuntu.mk
+++ b/rk356x/firefly-rk3568-ubuntu.mk
@@ -18,7 +18,7 @@ export RK_IDBLOCK_UPDATE=true
 # update spl
 export RK_LOADER_UPDATE_SPL=true
 #Set extboot
-export FF_EXTBOOT=true
+export FF_EXTBOOT=false
 
 # Set rootfs type, including ext2 ext4 squashfs
 export RK_ROOTFS_TYPE=ext4
```

然后重新烧录 `boot.img`。

5、烧录固件成功后重启即可正常启动：
```bash
root@firefly:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.8G     0  3.8G   0% /dev
tmpfs           769M  816K  768M   1% /run
/dev/nvme0n1p6  461M  363M   71M  84% /root-ro
/dev/nvme0n1p7  219G   69M  219G   1% /userdata
overlayroot     219G   69M  219G   1% /
tmpfs           3.8G     0  3.8G   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
tmpfs           769M     0  769M   0% /run/user/0
```

可以看到`userdata`分区挂载的设备为`/dev/nvme0n1p7`！

### SATA SSD 启动

SATA SSD 启动的操作步骤与 M.2 SSD 启动类似，不同点如下：

1、编译引导固件命令如下：
```bash
./make.sh firefly-rk3568-sata --spl-new
```

2、在 uboot shell 输入以下命令把系统固件烧录到 SATA SSD：
```bash
=> rockusb 0 scsi 0
```

### 没有贴 Nor Flash 的情况

由于 Maskrom 不支持从 SSD 加载引导固件，因此如果没有 Nor Flash 需要把引导固件烧录到 eMMC 存储器。

- 如果 eMMC 存储器本身有固件，那么直接进入 Loader 模式烧录`rk356x_spl_loader_*.bin`与`uboot.img`即可！
- 如果 eMMC 存储器本身没有固件，那么 Maskrom 模式是默认烧录到 eMMC 存储器的。