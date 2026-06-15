## 外部存储设备 rootfs 挂载

根文件系统除了可以使用在内部的 eMMC 中的，还可以使用外部存储设备的根文件系统，如 SD 卡，U 盘等。以下是以 SD 卡为例，在 Firefly-RK3399 设备上实现挂载外部存储设备的根文件系统。


### 在 SD 卡上建立分区

在 PC 机上插入 SD 卡，用 `gdisk` 工具分出一个装载根文件系统的分区：

```bash
# 用户请用 `fdisk -l` 查询 SD 卡设备名是什么，在用 gdisk 命令进入对应的设备
sudo gdisk /dev/sdb

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):

-------------------------------------------------------------------

# 输入 ? 查看帮助信息
# 按 p 显示 SD 卡当前已有分区，由于使用的 SD 卡暂未创建分区，所以没有查询到分区信息

Command (? for help): p
Disk /dev/sdb: 15278080 sectors, 7.3 GiB
Model: CARD-READER
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 5801AE61-92ED-42FB-A144-E522E8E15827
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 15278046
Partitions will be aligned on 2048-sector boundaries
Total free space is 15278013 sectors (7.3 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

-------------------------------------------------------------------

# 创建新的分区，请用户根据自身实际情况创建合适的分区大小给根文件系统

Command (? for help): n     # 输入 n 创建新分区
Partition number (1-128, default 1): 1      # 输入1创建第一个分区
First sector (34-15278046, default = 2048) or {+-}size{KMGTP}:   # 直接按回车，使用默认值
Last sector (2048-15278046, default = 15278046) or {+-}size{KMGTP}: +3G     # 创建分区大小为3G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):       # 按回车使用默认值
Changed type of partition to 'Linux filesystem'

-------------------------------------------------------------------

# 创建完成后，输入 i，可以查看分区的 unique GUID

Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: 6278AF5D-BC6B-4213-BBF6-47D333B5CB53   # 使用的是该分区的 unique GUID，记下前12位即可
First sector: 2048 (at 1024.0 KiB)
Last sector: 6293503 (at 3.0 GiB)
Partition size: 6291456 sectors (3.0 GiB)
Attribute flags: 0000000000000000
Partition name: 'Linux filesystem'

-------------------------------------------------------------------

# 输入 wq 保存修改并退出

Command (? for help): wq

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

```

把新创建的分区进行格式化。

```bash
# 注意 sdb1 表示 SD 卡对应分区设备名，请用户根据实际情况填入
sudo mkfs.ext4 /dev/sdb1

```

格式化完成后，使用 `dd` 命令，将制作好的根文件系统烧写到 SD 卡刚新建的分区中。（根文件系统的定制可以参考：[《Ubuntu 根文件系统定制》](custom_ubuntu_rootfs.md)）

```bash
# 用户请根据实际情况填写根文件系统镜像的路径和 SD 卡对应分区设备名
dd conv=fsync,notrunc if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### 挂载 SD 卡根文件系统

首先要修改设备树文件，打开对应的 dts 文件，在根节点下重写 `chosen` 节点下的 `bootargs` 参数的 `root` 值为写入了根文件系统的 SD 卡分区的 unique GUID：

```bash
# 将 root 的值改为获取到的 unique GUID 值的前 12 位
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B  rootfstype=ext4 rootwait";
};

```

修改完成后编译并烧录至设备中。

SD 卡的根文件系统烧录完成后，插入设备的 TF 卡槽中，开机即可进入到 SD 的根文件系统中。


注意事项：
*   在操作前请注意备份 U 盘或者 SD 卡内的重要文件，避免丢失；
*   操作时请确认自己的 SD 卡对应的设备名；
*   dts 文件中 `root` 参数的值使用的是 unique GUID，记下前12位即可。用户也可以根据 `gdisk` 的帮助信息自行修改这个值。