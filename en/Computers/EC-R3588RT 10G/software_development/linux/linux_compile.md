# Compile Linux firmware

<CodeBlockTabs defaultValue="k61">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="k510">kernel 5.10</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="k61">kernel 6.1</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="k510">
## Download Firefly_Linux_SDK [step]

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<Callout title="Compilation Notes" type="info">
	<font color=red>

**Attention:**

**1. SDK uses cross-compile, so download SDK to your X86_64 PC, do not download SDK to arm64 board.**

**2. Build environment needs to be Ubuntu18.04 (real PC or docker container), other versions may cause build failure.**

**3. Do not put SDK under shared directory of VM or non-English path.**

**4. Please get/build SDK as a normal user, root privilege are neither allowed nor required (except installing sth. with apt)**
	</font>
</Callout>

### Install Tools [step]
```
sudo apt update
sudo apt install -y repo git python
```

### Init Code Repository [step]

* Method One

Download via repo, you can choose to get full SDK or BSP:

```bash
mkdir ~/proj/rk3588_sdk/
cd ~/proj/rk3588_sdk/

## Full SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_release.xml

## BSP (Only include some basic repositories and compile tools(Only for ubuntu & debian))
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk3588_linux_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Linux SDK](https://community.t-firefly.com/en/doc/download/233), follow the README document:

```bash
└── rk3588_linux_release_xxx
    ├── linux_sdk_tar
    │   ├── rk3588_linux_release_xxx.sdk.split00
    │   ├── rk3588_linux_release_xxx.sdk.split01
    │   ├── rk3588_linux_release_xxx.sdk.split02
    │   ├── rk3588_linux_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

### Sync Code [step]

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3588_sdk

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```
	**Linux SDK Configuration introduction**

	<Accordions>

	<Accordion title="Directory" type="single" id="sdk_dir">

	```bash
	$ tree -L 1
	.
	├── app
	├── buildroot # Buildroot root filesystem build directory
	├── build.sh -> device/rockchip/common/build.sh # Compile script
	├── device # Compile related configuration files
	├── docs # Documentation
	├── envsetup.sh -> buildroot/build/envsetup.sh
	├── external
	├── kernel
	├── Makefile -> buildroot/build/Makefile
	├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh # Link script
	├── prebuilts # Cross compilation toolchain
	├── rkbin
	├── rkflash.sh -> device/rockchip/common/rkflash.sh # Flash script
	├── tools # Tools directory
	├── u-boot
	```

	</Accordion>

	<Accordion title="Introduction to configuration files" type="single" id="sdk_config">

	In the `device/rockchip/rk3588/` directory, there are configuration files (xxxx.mk) for different board types, which are used to manage the compilation configuration of each project of the SDK. The relevant configuration introduction:

	```bash
	# Target arch
	export RK_ARCH=arm64
	# Uboot defconfig
	export RK_UBOOT_DEFCONFIG=xxxx_defconfig
	# Kernel defconfig
	export RK_KERNEL_DEFCONFIG=xxxx_defconfig
	# Kernel defconfig fragment
	export RK_KERNEL_DEFCONFIG_FRAGMENT=xxxx.config
	# Kernel dts
	export RK_KERNEL_DTS=roc-rk3588-rt.dts
	# parameter for GPT table
	export RK_PARAMETER=parameter-xxxx.txt
	# rootfs image path
	export RK_ROOTFS_IMG=ubuntu_rootfs/rootfs.img
	```

	</Accordion>

	<Accordion title="Partition table" type="single" id="sdk_partition">

	**parameter**

	The parameter.txt file contains the partition information of the firmware. Take parameter-ubuntu-fit.txt as an example:

	path: `device/rockchip/rk3588/parameter-ubuntu-fit.txt`

	```bash
	FIRMWARE_VER: 1.0
	MACHINE_MODEL: RK3588
	MACHINE_ID: 007
	MANUFACTURER: RK3588
	MAGIC: 0x5041524B
	ATAG: 0x00200800
	MACHINE: 0xffffffff
	CHECK_MASK: 0x80
	PWR_HLD: 0,0,A,0,1
	TYPE: GPT
	CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00020000@0x00008000(boot:bootable),0x00040000@0x00028000(recovery),0x00010000@0x00068000(backup),0x00c00000@0x00078000(rootfs),0x00040000@0x00c78000(oem),-@0x00cb8000(userdata:grow)
	uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
	```

	<Callout title="Parameter Description" type="info">
	The CMDLINE attribute is where we are concerned. Take uboot as an example. In 0x00002000@0x00004000(uboot), 0x00004000 is the starting position of the uboot partition, 0x00002000 is the size of the partition, and so on.
	</Callout>

	**package-file**

	The package-file is used to determine the required partition image and image path when packaging the firmware, and it needs to be consistent with the parameter.txt.

	path: `tools/linux/Linux_Pack_Firmware/rockdev/rk3588-package-file`

	```bash
	# NAME          Relative path
	#
	#HWDEF          HWDEF
	package-file    package-file
	bootloader      Image/MiniLoaderAll.bin
	parameter       Image/parameter.txt
	uboot           Image/uboot.img
	misc            Image/misc.img
	boot            Image/boot.img
	recovery        Image/recovery.img
	rootfs          Image/rootfs.img
	userdata        RESERVED
	backup          RESERVED
	```

	</Accordion>

	</Accordions>
## Compile firmware [step]

  <CodeBlockTabs defaultValue="debian">
    <CodeBlockTabsList>
    </CodeBlockTabsList>




  </CodeBlockTabs>

  </CodeBlockTab>

  <CodeBlockTab value="k61">
## Download the SDK [step]

Please contact sales@t-firefly.com to get SDK download link and read the readme file

<Callout title="Compilation Notes" type="info">
	<font color="red">

**Notice:**

**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**

**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**

**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**

**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**
	</font>
</Callout>

	**SDK Introduction**

	<Accordions>

	<Accordion title="SDK Structure" type="single" id="sdk61_dir">

	```bash
	$ tree -L 1
	.
	├── app
	├── buildroot                                               # buildroot
	├── build.sh -> device/rockchip/common/scripts/build.sh     # building script
	├── device                                                  # building system, includes building configs
	├── docs                                                    # development documents
	├── external                                                # some components
	├── kernel -> kernel-6.1
	├── kernel-6.1                                              # linux kernel
	├── Makefile -> device/rockchip/common/Makefile
	├── prebuilt_rootfs                                         # used to place pre-built rootfs
	├── prebuilts                                               # used to place cross-build toolchain
	├── rkbin
	├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
	├── tools                                                   # some tools
	└── u-boot                                                  # uboot
	```

	</Accordion>

	<Accordion title="About Configurations" type="single" id="sdk61_config">

	There are many config files (xxxx_defconfig) under `device/rockchip/rk3588/`, these files control the building system.

	And their names will be like:
	```
	<vendor>_<chip>_<model>-<extra>_<OS>_defconfig

	vendor: vendor of the product
	chip: soc the product used
	model: product model, means this config file is for that specific product.
	extra: some extra features, could be empty
	OS: the operating system that will be running in product
	```

	About config file contents, take firefly_rk3588_itx-3588j_debian_defconfig as an example:
	```bash
	RK_KERNEL_DTS_NAME="rk3588-firefly-itx-3588j"                    # set the dts to be used during kernel compilation
	RK_KERNEL_CFG_FRAGMENTS="firefly-linux.config"                   # set the config fragments to be used during kernel compilation
	RK_UBOOT_CFG_FRAGMENTS="firefly-linux"                           # set the config fragments to be used during uboot compilation
	RK_USE_FIT_IMG=y
	RK_BOOT_FIT_ITS_NAME="bootramdisk.its"
	RK_PARAMETER="parameter-debian-fit.txt"                          # set the partition parameter
	RK_PRODUCT_MODEL="ITX-3588J"                                     # product name/model
	PREBUILT_ROOTFS_IMG="prebuilt_rootfs/rk3588_debian_rootfs.img"   # set the prebuilt rootfs to be used
	RK_MISC_RECOVERY=y
	RK_RECOVERY_RAMDISK="rk3588-recovery-arm64.cpio.gz"
	RK_RAMDISK_IMG="kernel/ramdisk.img"
	USE_EXTBOOT=y                                                    # use exlinux style boot.img
	RK_EXTRA_PARTITION_NUM=1
	```

	</Accordion>

	<Accordion title="Partitions" type="single" id="sdk61_partition">

	Parameter file records the firmware partitions, for example `device/rockchip/rk3588/parameter-xxxxxx-fit.txt`

	```bash
	FIRMWARE_VER: 1.0
	MACHINE_MODEL: RK3588
	MACHINE_ID: 007
	MANUFACTURER: RK3588
	MAGIC: 0x5041524B
	ATAG: 0x00200800
	MACHINE: 0xffffffff
	CHECK_MASK: 0x80
	PWR_HLD: 0,0,A,0,1
	TYPE: GPT
	GROW_ALIGN: 0
	CMDLINE: mtdparts=:0x00002000@0x00004000(uboot),0x00002000@0x00006000(misc),0x00040000@0x00008000(boot:bootable),0x00040000@0x00048000(recovery),0x00010000@0x00088000(backup),0x01c00000@0x00098000(rootfs),-@0x01c98000(userdata:grow)
	uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
	uuid:boot=7A3F0000-0000-446A-8000-702F00006273
	```
	<Callout title="Parameter Description" type="info">
	CMDLINE is what we need to focus on, in "0x00002000@0x00004000(uboot)", 0x00004000 is the start position of uboot, and 0x00002000 is the size of uboot partition. Start position + Partition size = Start position of the next partition. All partitions follow the same rule.

	And the unit of these hex numbers is "block", each block is 512 Byte, which means uboot partition size 0x00002000 is 8192 block = 8192 x 512 / 1024 / 1024 = 4 MiB
	</Callout>

	</Accordion>

	</Accordions>
## Compile firmware [step]

  <CodeBlockTabs defaultValue="debian">
    <CodeBlockTabsList>
    </CodeBlockTabsList>



  </CodeBlockTabs>

  </CodeBlockTab>
</CodeBlockTabs>