## Build Ubuntu firmware

This chapter introduces the compilation process of Ubuntu firmware.

### Build Linux-SDK

#### Precompile Configuration

There are configuration files for different board in `device/rockchip/rk3288/`, select the configuration file:

##### HDMI

```bash
./build.sh aio-3288j-ubuntu.mk
```


##### LVDS DM-M10R800

```bash
./build.sh aio-3288j-lvds-ubuntu.mk
```


The configuration file will be connected to `device/rockchip/.BoardConfig.mk`, check the file to verify whether the configuration is successful.

Configruation content:

```bash
# Target arch
export RK_ARCH=arm                                              # 32-bit ARM
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3288                        # u-boot configuration
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig              # kernel configuration
# Kernel dts
export RK_KERNEL_DTS=rk3288-firefly-aio                              # dts file
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt                        # partition table
# rootfs image path
export RK_ROOTFS_IMG=ubuntu_rootfs/rk3288_ubuntu_rootfs.img     # filesystem path
```

#### Partial compilation

* u-boot

```bash
./build.sh uboot
```

* kernel

```bash
./build.sh kernel
```

* recovery

```bash
./build.sh recovery
```

#### Download Ubuntu filesystem

* Download: [Ubuntu rootfs(32-bit)](http://en.t-firefly.com/doc/download/16.html#other_122)，put in SDK path

* Unzip

```bash
7z x ubuntu-armhf-rootfs.7z
```

* Move filesystem to `ubuntu_rootfs/`

```bash
mkdir ubuntu_rootfs
mv ubuntu-armhf-rootfs.img ubuntu_rootfs/rk3288_ubuntu_rootfs.img
```

#### Pack the firmware

Update each part of the `.img` link to the directory `rockdev/`:

```bash
./mkfirmware.sh
```

Pack the firmware, the firmware will be saved to the directory `rockdev/pack/`.

```bash
./build.sh updateimg
```

#### Automatic compilation

The automatic compilation will perform the above compilation and packaging operations to generate complete firmware.

```bash
./build.sh
```

### Partition table

#### parameter

The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu.txt as an example:

path: `device/rockchip/rk3288/parameter-ubuntu.txt`

```bash
FIRMWARE_VER: 8.1
MACHINE_MODEL:RK3288
MACHINE_ID:007
MANUFACTURER:RK3288
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3288
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00600000@0x0005a000(rootfs),-@0x0065a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```

The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.

#### package-file

The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3288-ubuntu-package-file`

```bash
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
misc            Image/misc.img
recovery        Image/recovery.img
rootfs          Image/rootfs.img
userdata:grow RESERVED
backup          RESERVED
```