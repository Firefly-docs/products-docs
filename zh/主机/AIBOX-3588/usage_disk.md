# U 盘、TF 卡、SSD 挂载
AIBOX-3588  支持一系列的存储设备接口（PCIE，SATA，USB，TF卡）。

当插入存储设备时，存储设备会被识别为 `/dev/sdb1`，`/dev/nvme0n1p1` 或者是 `/dev/mmcblkp1` 类似节点，与桌面 PC Linux 环境下相同。

文件系统支持 FAT、FAT32、EXT2/3/4、NTFS。AIBOX-3588 不支持自动挂载，所以需要手工进行 `mount` 挂载。完成数据写入后，请及时使用 `sync` 或 `umount` 操作，关机时请使用 `sudo poweroff` 命令，避免暴力下电关机，以免数据丢失。

## U 盘（SATA 硬盘）挂载

插入 U 盘（SATA 硬盘），使用 `dmesg` 可知道 U 盘（SATA 硬盘）对应的 `sd` 设备：

```shell
[15460.953423] [5]  sdb: sdb1
```

挂载 U 盘（SATA 硬盘）：

```shell
# 创建挂载目录
mkdir disk
# 挂载
sudo mount /dev/sdb1 disk
# 查看 U 盘（SATA 硬盘）内的文件
ls disk
```

## TF 卡挂载

TF 卡挂载与 U 盘是类似的，使用 `dmesg` 可知道 TF 卡对应的 `mmcblk` 设备：

```shell
[16220.776440] [4]  mmcblk1: p1
```

挂载 TF 盘：

```shell
# 创建挂载目录
mkdir media
# 挂载
sudo mount /dev/mmcblk1p1 media
# 查看 TF 卡内的文件
ls media
```
## PCIE SSD 挂载

一般可挂载的 PCIE 节点为：/dev/nvme0n1p1。  

挂载 PCIE SSD：

```shell
# 创建挂载目录
mkdir nvme_dir
# 挂载
sudo mount /dev/nvme0n1p1 nvme_dir
# 查看 PCIE SSD的文件
ls nvme_dir
```