# 固件更新

## 更新 SDK
遵循 NVIDIA 官方的方式，采用 `Recovery` 线烧模式，用于更新固件的 PC 最好是 **Ubuntu22.04** 及以上版本的系统，并做好以下准备工作（以 `r36.3` SDK 版本为例）:

1.下载官方 SDK 包（可直接在 Firefly [下载页面](https://www.t-firefly.com/doc/download/266.html)进行下载），一共有 4 个压缩包：

* `jetson_linux_r36.3.0_aarch64.tbz2`                           # Linux_for_Tegra 基础包
* `tegra_linux_sample-root-filesystem_r36.3.0_aarch64.tbz2`    # 根文件系统
* `public_sources.tbz2`                                         # SDK 源码 （包含内核源码以及 BSP 设备树）
* `aarch64--glibc--stable-2022.08-1.tar.bz2`                    # 交叉编译工具链


2.解压官方 SDK 包，形成 `Linux_for_Tegra` 目录树结构：

```
$ mkdir jetson-orin && cd jeton-orin
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

3.下载 Firefly SDK 包（基于 NVIDIA 官方 SDK 所做修改）并解压到与 `Linux_for_Tegra` 同级目录,如下图所示：

```
$ tree -L 1 ./
./
├── extract_repositories.sh
├── Linux_for_Tegra
└── patch
```

4.覆盖 SDK：

```
./extract_repositories.sh patch/repos.txt
```

覆盖 SDK 后，走官方的编译烧写流程即可。

## 烧写固件

NVIDIA 官方更新固件的方式主要依赖于线烧，连接用于烧录的 PC 以及 Orin 设备（Type-C），按住 `Recovery` [按键](interface_definition.html#zheng-ji-jie-kou-ding-yi)并接入电源，设备便进入烧录模式。


烧录脚本有两种，分别对应不同场景下的烧录：

* `flash.sh`
* `l4t_initrd_flash.sh`

烧录的时候对于特定的脚本（例如 `l4t_initrd_flash.sh`）可以进行适当的传参：
* `BOARDID=3767`                                 # 指定底板，默认 Firefly 为 3767
* `BOARDSKU=0001`                                # 指定核心板规格，8G 内存的 Orin Nano 为 0005， 16G 内存的 Orin NX 为 0001
* `ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo` # 表示 bootload 中的 UEFI 优先选择 PCIE 启动

下面**以烧录 16G Orin NX 为例子：**

需要注意的是，以下烧录步骤均包含了固件的编译：

```shell
cd path/to/Linux_for_Tegra # 先切到烧录目录
```  

* 只烧录 bootloader（往 QSPI 里面进行烧录，类似于 PC 的 BIOS，这一步如果不是对 Orin 设备非常熟悉，**请不要执行**，否则设备会变砖）  

```shell
sudo ./flash.sh --no-systemimg -c bootloader/generic/cfg/flash_t234_qspi.xml \
    jetson-orin-nano-devkit <rootdev> # <rootdev> 可以是 internal 或者是 external
```

* 全烧录（bootloader + 系统镜像）

系统镜像往 PCIE 设备里面烧录：  

```shell
sudo BOARDID=3767 BOARDSKU=0001 ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo \
    ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 \
    -c tools/kernel_flash/flash_l4t_t234_nvme.xml -p "-c bootloader/generic/cfg/flash_t234_qspi.xml" \
    --showlogs --network usb0 jetson-orin-nano-devkit internal
```

### 其他

同时 NVIDIA 同时提供类似于磁盘刻录的烧录方式，即烧录的 PC 主机直接与 PCIE 相连接,直接把系统刻录到 PCIE 设备里面，该方法适用于一些系统盘可拆卸的 Orin 设备:

比较重要的参数是：

* `--external-device sdb` 表示需要进行磁盘刻录的块设备（如果用的是 PCIE 转 USB 设备连接的 PC，那么显示的就是 `sdx` 设备）

```
sudo BOARDID=3767 BOARDSKU=0001 ADDITIONAL_DTB_OVERLAY_OPT=BootOrderNvme.dtbo \
    ./tools/kernel_flash/l4t_initrd_flash.sh   -c tools/kernel_flash/flash_l4t_external.xml  \
    --external-device sdb --direct sdb jetson-orin-nano-devkit external
```