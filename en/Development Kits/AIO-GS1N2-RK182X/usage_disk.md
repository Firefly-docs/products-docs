# USB drive, TF card mount

When a USB flash drive or TF card is inserted, the storage device will be recognized as a node similar to `/dev/sdb1` or `/dev/mmcblkp1`, which is the same as in the desktop PC Linux environment.

The file system supports FAT, FAT32, EXT2/3/4, NTFS. RK182X开发套件 does not support automatic mounting, so `mount` needs to be manually mounted. After completing the data writing, please use `sync` or `umount` in time, and please use the `sudo poweroff` command when shutting down to avoid violent power-off and shutdown to avoid data loss.

## USB drive mount

Insert the USB drive, use `dmesg` to know the `sd` device corresponding to the USB drive:

```shell
[15460.953423] [5] sdb: sdb1
```

Mount the USB drive:

```shell
# create mount directory
mkdir disk
# mount
sudo mount /dev/sdb1 disk
# View the files in the USB drive
ls disk
```

## TF card mount

The TF card mounting is similar to the USB drive, use `dmesg` to know the `mmcblk` device corresponding to the TF card:

```shell
[16220.776440] [4] mmcblk1: p1
```

Mount TF disk:

```shell
# create mount directory
mkdir media
# mount
sudo mount /dev/mmcblk1p1 media
# View the files in the TF card
ls media
```