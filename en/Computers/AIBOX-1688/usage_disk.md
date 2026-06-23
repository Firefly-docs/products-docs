# USB Drive, TF Card, SSD Mounting
AIBOX-1688 supports a range of storage device interfaces (PCIE, SATA, USB, TF card).

When a storage device is inserted, it will be recognized as `/dev/sdb1`, `/dev/nvme0n1p1`, or `/dev/mmcblkp1`, similar to nodes in a desktop PC Linux environment.

The file system supports FAT, FAT32, EXT2/3/4, and NTFS. The AIBOX-1688 does not support automatic mounting, so manual `mount` is required. After completing data writing, please promptly use `sync` or `umount` operations. When shutting down, please use the `sudo poweroff` command to avoid a forced power-off, which may lead to data loss.

## USB Drive (SATA Hard Drive) Mounting

Insert the USB drive (SATA hard drive) and use `dmesg` to find the corresponding `sd` device:

```shell
[15460.953423] [5]  sdb: sdb1
```

Mount the USB drive (SATA hard drive):

```shell
# Create mount directory
mkdir disk
# Mount
sudo mount /dev/sdb1 disk
# View files in the USB drive (SATA hard drive)
ls disk
```

## TF Card Mounting

Mounting the TF card is similar to the USB drive; use `dmesg` to find the corresponding `mmcblk` device:

```shell
[16220.776440] [4]  mmcblk1: p1
```

Mount the TF card:

```shell
# Create mount directory
mkdir media
# Mount
sudo mount /dev/mmcblk1p1 media
# View files in the TF card
ls media
```

## PCIE SSD Mounting

Generally, the mountable PCIE node is: /dev/nvme0n1p1.

Mount the PCIE SSD:

```shell
# Create mount directory
mkdir nvme_dir
# Mount
sudo mount /dev/nvme0n1p1 nvme_dir
# View files in the PCIE SSD
ls nvme_dir
```