## External storage device rootfs mount

In addition to the root file system used in the internal eMMC, you can also use the root file system of external storage devices, such as SD cards, U disks, etc. The following takes SD card as an example to realize the root file system of mounting external storage device on Firefly-RK3399 device.


### Create a partition on the SD card

Insert an SD card into the PC, and use the `gdisk` tool to separate a partition for mounting the root file system:

```bash
# Users, please use `fdisk -l` to query what the SD card device name is, and use the gdisk command to enter the corresponding device
sudo gdisk /dev/sdb

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):

-------------------------------------------------- -----------------

# Enter? View help information
# Press p to display that the SD card currently has partitions. Since the used SD card has not yet created partitions, no partition information can be found

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

Number Start (sector) End (sector) Size Code Name

-------------------------------------------------- -----------------

# Create a new partition, please create a suitable partition size for the root file system according to your actual situation

Command (? for help): n # Enter n to create a new partition
Partition number (1-128, default 1): 1 # Enter 1 to create the first partition
First sector (34-15278046, default = 2048) or {+-}size{KMGTP}: # Press enter directly, use the default value
Last sector (2048-15278046, default = 15278046) or {+-}size{KMGTP}: +3G # Create partition size of 3G
Current type is'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): # Press Enter to use the default value
Changed type of partition to'Linux filesystem'

-------------------------------------------------- -----------------

# After creation, enter i to view the unique GUID of the partition

Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: 6278AF5D-BC6B-4213-BBF6-47D333B5CB53 # Use the unique GUID of the partition, just write down the first 12 digits
First sector: 2048 (at 1024.0 KiB)
Last sector: 6293503 (at 3.0 GiB)
Partition size: 6291456 sectors (3.0 GiB)
Attribute flags: 0000000000000000
Partition name:'Linux filesystem'

-------------------------------------------------- -----------------

# Enter wq to save changes and exit

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

Format the newly created partition.

```bash
# Note sdb1 represents the device name of the partition corresponding to the SD card, please fill in according to the actual situation
sudo mkfs.ext4 /dev/sdb1

```

After formatting is complete, use the `dd` command to burn the created root file system to the newly created partition of the SD card. (For the customization of the root file system, please refer to *[Ubuntu Rootfs Customization](custom_ubuntu_rootfs.md)*)

```bash
# User please fill in the path of the root file system mirror and the device name of the corresponding partition of the SD card according to the actual situation
dd conv=fsync,notrunc if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### Mount SD card root file system

First, modify the device tree file, open the corresponding dts file, and rewrite the `root` value of the `bootargs` parameter under the `chosen` node under the root node to the unique GUID of the SD card partition written in the root file system:

```bash
# Change the value of root to the first 12 digits of the unique GUID value obtained
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B rootfstype=ext4 rootwait";
};

```

Compile and burn to the device after modification.

After the root file system of the SD card has been burned, insert it into the TF card slot of the device and boot into the root file system of the SD card.


Precautions:
* Please pay attention to back up important files in the U disk or SD card before operation to avoid loss;
* Please confirm the device name corresponding to your SD card during operation;
* The value of the `root` parameter in the dts file uses unique GUID, just write down the first 12 digits. The user can also modify this value according to the help information of `gdisk`.