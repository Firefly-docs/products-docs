# Firmware Update

## Update SDK
Follow the official NVIDIA method, using the `Recovery` line burning mode. It is best to use a PC with **Ubuntu 22.04** or higher for updating the firmware, and prepare as follows (using `r36.3` SDK version as an example):

1. Download the official SDK package (can be downloaded directly from the Firefly [download page](https://www.t-firefly.com/doc/download/266.html)), which includes 4 compressed packages:

* `jetson_linux_r36.3.0_aarch64.tbz2`                           # Linux_for_Tegra base package
* `tegra_linux_sample-root-filesystem_r36.3.0_aarch64.tbz2`    # Root filesystem
* `public_sources.tbz2`                                         # SDK source code (includes kernel source and BSP device tree)
* `aarch64--glibc--stable-2022.08-1.tar.bz2`                    # Cross-compilation toolchain

2. Unzip the official SDK package to create the `Linux_for_Tegra` directory structure:

```
$ mkdir jetson-orin && cd jetson-orin
$ cp jetson_linux_r36.3.0_aarch64.tbz2 ./ 
$ cp tegra_linux_sample-root-filesystem_r36.3.0_aarch64.tbz2  ./ 
$ cp public_sources.tbz2   ./

$ tar xf jetson_linux_r36.3.0_aarch64.tbz2
$ sudo tar xpf tegra_linux_sample-root-filesystem_r36.3.0_aarch64.tbz2 -C Linux_for_Tegra/rootfs/
$ cd Linux_for_Tegra/
$ sudo ./tools/l4t_flash_prerequisites.sh
$ sudo ./apply_binaries.sh

$ cd -
$ tar xf public_sources.tbz2 -C ./Linux_for_Tegra/..
$ cd  ./Linux_for_Tegra/source
$ tar xf kernel_src.tbz2
$ tar xf kernel_oot_modules_src.tbz2
$ tar xf nvidia_kernel_display_driver_source.tbz2
```

3. Download the Firefly SDK package (modified based on the official NVIDIA SDK) and unzip it to the same level directory as `Linux_for_Tegra`, as shown below:

```
$ tree -L 1 ./
./
├── extract_repositories.sh
├── Linux_for_Tegra
└── patch
```

4. Overwrite the SDK:

```
./extract_repositories.sh patch/repos.txt
```

After overwriting the SDK, follow the official compilation and flashing process.

## Flashing Firmware

NVIDIA's official method for updating firmware mainly relies on flashing, connecting the PC used for flashing and the Orin device (Type-C), holding down the `Recovery` [button](interface_definition.html#zheng-ji-jie-kou-ding-yi) and plugging in the power, the device will enter flashing mode.

There are two types of flashing scripts, corresponding to different scenarios:

* `flash.sh`
* `l4t_initrd_flash.sh`

When flashing, specific parameters can be passed for certain scripts (e.g., `l4t_initrd_flash.sh`):
* `BOARDID=3767`                                 # Specify the baseboard, default Firefly is 3767
* `BOARDSKU=0001`                                # Specify the core board specifications, 8G memory Orin Nano is 0005, 16G memory Orin NX is 0001
* `ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo` # Indicates UEFI boot order preference for PCIE boot

Below is **an example of flashing the 16G Orin NX:**

It is important to note that the following flashing steps include firmware compilation:

```shell
cd path/to/Linux_for_Tegra # First switch to the flashing directory
```  

* Flash only the bootloader (flashing into QSPI, similar to PC BIOS; if you are not very familiar with Orin devices, **please do not execute this step**, otherwise the device may become bricked)  

```shell
sudo ./flash.sh --no-systemimg -c bootloader/generic/cfg/flash_t234_qspi.xml \
    jetson-orin-nano-devkit <rootdev> # <rootdev> can be internal or external
```

* Full flashing (bootloader + system image)

Flash the system image into the PCIE device:  

```shell
sudo BOARDID=3767 BOARDSKU=0001 ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo \
    ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 \
    -c tools/kernel_flash/flash_l4t_t234_nvme.xml -p "-c bootloader/generic/cfg/flash_t234_qspi.xml" \
    --showlogs --network usb0 jetson-orin-nano-devkit internal
```

### Others

NVIDIA also provides a method similar to disk burning, where the flashing PC directly connects to the PCIE, directly burning the system onto the PCIE device. This method is suitable for some removable system disk Orin devices:

The more important parameter is:

* `--external-device sdb` indicates the block device to be burned (if using a PC connected via a PCIE to USB device, it will show as `sdx` device)

```
sudo BOARDID=3767 BOARDSKU=0001 ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo \
    ./tools/kernel_flash/l4t_initrd_flash.sh   -c tools/kernel_flash/flash_l4t_external.xml  \
    --external-device sdb --direct sdb jetson-orin-nano-devkit external
```