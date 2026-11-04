# Compile Android firmware

<CodeBlockTabs defaultValue="a12">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="a12">Android 12</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="a14">Android 14</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="a12">

## Download Android12.0 SDK [step]

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3588_android12_git_20240704](https://community.t-firefly.com/en/doc/download/201)

* After downloading, verify the MD5 code:
   ```
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.001
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.002
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.003
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.004
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.005
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.006
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.007

    1ce2a5ab4e3a9f0a4d6d8cd47ca4981d  firefly_rk3588_android12_git_20240704.7z.001
    553fb5ee2cc5623d97bfb19ba299605b  firefly_rk3588_android12_git_20240704.7z.002
    cb9cce227a79356fa57c0781a426fd89  firefly_rk3588_android12_git_20240704.7z.003
    1461a09bacbef0cb3c9a9008e8483e93  firefly_rk3588_android12_git_20240704.7z.004
    d154b39087bbf410a7973da97a6a9ecb  firefly_rk3588_android12_git_20240704.7z.005
    7cfc42443caa57cb4cf474bf3e78b5ce  firefly_rk3588_android12_git_20240704.7z.006
    9ad4374d89212a882dc00541769732f8  firefly_rk3588_android12_git_20240704.7z.007

    ```
* After confirming that it is correct, we can unzip:

   ```
    cd ~/proj/
    7z x ./firefly_rk3588_android12_git_20240704.7z.001 -oRK3588_Android12.0
    cd ./RK3588_Android12.0
    git reset --hard
   ```

<Callout title="Note" type="warn">
	To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.
</Callout>

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK3588_Android12.0

2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3588-android12.0-bundle.7z  -r -o. && mv rk3588-android12.0-bundle .bundle

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android12.0 Bundle](https://community.t-firefly.com/en/doc/download/201)。



<Callout title="git clone remote bundle repository failed" type="info">
	When updating the SDK, the following error may occur while cloning the remote bundle repository:

	```
	$ git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle
	Cloning into '.bundle'...
	remote: Enumerating objects: 20, done.
	remote: Counting objects: 100% (20/20), done.
	remote: Compressing objects: 100% (18/18), done.
	error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
	fatal: the remote end hung up unexpectedly
	fatal: early EOF
	fatal: unpack-objects failed
	```

	The buffer size is insufficient. Enlarge it with the following command and retry:

	```
	git config --global https.postBuffer 1048576000
	```
</Callout>
## Core-3588SG product compilation method [step]

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">Full compilation</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">Step by step</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA Compilation</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
	### The overall compilation [step]

	#### Public Firmware Compilation
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
  </CodeBlockTab>

  <CodeBlockTab value="partial">
	### Step by step to compile [step]

	1. Compile the kernel:

	```
	cd ~/proj/RK3588_Android12.0/kernel-5.10
	export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
	msk ARCH=arm64   BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8
	```

	2. Compile uboot:

	```
	cd ~/proj/RK3588_Android12.0/u-boot/
	./make.sh rk3588
	```

	3. Compile Android：

	```
	cd ~/proj/RK3588_Android12.0/
	source build/envsetup.sh
	lunch rk3588s_firefly_aio_3588sg-userdebug
	make installclean
	make -j8
	./mkimage.sh
	```

	### Packaged into unified firmware update.img [step]

	After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
	After packaging, it will be in rockdev/Image-rk3588s_firefly_aio_3588sg/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

	It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
	date.img and store it in rockdev \ Image directory.
### Build outputs [step]

	When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev/Image-rk3588s_firefly_aio_3588sg/. The following lists the image files used by general firmware:

	| Image | Instruction |
	|-------|-------------|
	| boot.img | including ramdis、kernel、dtb |
	| dtbo.img | Device Tree Overlays |
	| MiniLoaderAll.bin | including first level loader |
	| misc.img | including recovery-wipe boot symbol information, after flashing it will enter recovery |
	| parameter.txt | including partition information |
	| recovery.img | including recovery-ramdis、kernel、dtb |
	| super.img | including the contents of odm、vendor、system partitions |
	| uboot.img | including uboot image |
	| vbmeta.img | including avb verification information, used for AVB verification |
	| update.img | including the above img files to be flashed, can be used for the tool to directly flash the whole image package |

	For details about how to upgrade a partition image file, see [USB Firmware Upgrade](../../firmware_upgrade/upgrade_firmware.md).
  </CodeBlockTab>

  <CodeBlockTab value="ota">
	## OTA Compilation 

	### Firefly full OTA compilation and upgrade
	#### Add the -o parameter for OTA full compilation
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug -o
	```

	#### Generate the full package path
	```
	out/target/product/rk3588s_firefly_aio_3588sg/rk3588s_firefly_aio_3588sg-ota-eng.xxx.zip 
	```
	#### Generate the base material package path
	```
	out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip
	```
	Copy the base material package and rename it, mainly for subcontracting upgrades.
	```
	cp out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip ~/ota/v1.zip 
	```

	#### Package compiled firmware and upgrades
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
	In the `rockdev/Image-rk3588s_firefly_aio_3588sg/` directory, find the corresponding firmware and burn it to the machine, then the machine can carry out the full package upgrade and the difference package upgrade,put 'update.zip' into /sdcard/ and restart the full package as prompted.

	### Generate the incremental OTA package
	#### Generate a new base material package 
	The v1.zip basic material package has been made, and another v2.zip material package needs to be made to generate the difference package. You need to modify the kernel DTS code or Android code, and then run the command again.

	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug -o
	```

	Regenerate a new base material package that we can copy.
	```
	 cp out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip  ~/ota/v2.zip
	```
	#### Generate the incremental OTA package command
	```
	out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
	```
	#### Upgrade OTA incremental OTA package
	Put update.zip to the machine sdcard directory reboot, it will automatically recognize, and then click the steps to complete the upgrade.

	### OTA server setup

	Refer to [OTA Documentation](https://community.t-firefly.com/en/doc/download/164) to setup the sever.

</CodeBlockTab>
</CodeBlockTabs>

<Callout title="Some Introduction about Compiling" type="info">
	**Android12.0 can’t be upgraded directly kernel.img and resource.img**

	`kernel.img` and `resource.img` for Android12.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev/Image-rk3588s_firefly_aio_3588sg/ directory.

	**Compiling kernel generation separately boot.img**

	Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

	```
	cd ~/proj/RK3588_Android12.0/kernel-5.10
	export PATH=../prebuilts/clang/host/linux-x86/clang-r416183b/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64  firefly_defconfig android-11.config pcie_wifi.config
	msk ARCH=arm64   BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8
	```

	After compiling, `boot.img` in the kernel directory can be directly upgrade.
</Callout>

  </CodeBlockTab>

  <CodeBlockTab value="a14">

## Download Android14.0 SDK [step]

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3588_android14_git_20240822](https://community.t-firefly.com/en/doc/download/201)

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.001
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.002
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.003
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.004
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.005
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.006
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.007
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.008
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.009
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.010
    $ md5sum /path/to/firefly_rk3588_android14_git_240822.7z.011

    c9cc4863a3ad3b48f3c74d1f061d0438  firefly_rk3588_android14_git_240822.7z.001
    e81083e1b9b435e4e666f7d08dcd1bd6  firefly_rk3588_android14_git_240822.7z.002
    49b7aace84ab01b9a014b410fa26f42f  firefly_rk3588_android14_git_240822.7z.003
    1632fc98d881d5f4a8ed953afc9c73b1  firefly_rk3588_android14_git_240822.7z.004
    f581633cefdef6103de584501612e5dd  firefly_rk3588_android14_git_240822.7z.005
    f0e591f54a8969755775642567dabb17  firefly_rk3588_android14_git_240822.7z.006
    ee3d00d688ac0ad2bb43f8fe73f354bd  firefly_rk3588_android14_git_240822.7z.007
    09dc458646cce9f53a792d68e0ece3ec  firefly_rk3588_android14_git_240822.7z.008
    add00c4da466d50782ea5dba279c72d1  firefly_rk3588_android14_git_240822.7z.009
    51438ba104f0a67fd4fa09b5fee0d301  firefly_rk3588_android14_git_240822.7z.010
    b733f1d13c070862b5af6ce71052c020  firefly_rk3588_android14_git_240822.7z.011

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android14.0_git_20240822.7z.001 -oRK3588_Android14.0
    cd ./RK3588_Android14.0
    git reset --hard
    ```

<Callout title="Note" type="warn">
	To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.
</Callout>

#### Second, Update SDK
Note: Be sure to update the remote warehouse after decompression. The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/RK3588_Android14.0

2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3588-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3588-android14.0-bundle.7z  -r -o. && mv rk3588-android14.0-bundle .bundle

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://community.t-firefly.com/en/doc/download/201)。



<Callout title="git clone remote bundle repository failed" type="info">
	When updating the SDK, the following error may occur while cloning the remote bundle repository:

	```
	$ git clone https://gitlab.com/T-Firefly/rk3588-Android14.0-bundle.git .bundle
	Cloning into '.bundle'...
	remote: Enumerating objects: 20, done.
	remote: Counting objects: 100% (20/20), done.
	remote: Compressing objects: 100% (18/18), done.
	error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
	fatal: the remote end hung up unexpectedly
	fatal: early EOF
	fatal: unpack-objects failed
	```

	The buffer size is insufficient. Enlarge it with the following command and retry:

	```
	git config --global https.postBuffer 1048576000
	```
</Callout>
## Core-3588SG product compilation method [step]

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">Full compilation</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">Step by step</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA Compilation</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
	### The overall compilation [step]

	#### Default Firmware Compilation
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```

	#### 10.1‘ MIPI DSI0 Firmware Compilation：
	```
	./FFTools/make.sh -d  -j8 -l rk3588s_firefly_aio_3588sg_mipi-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg_mipi-userdebug
	```
	<!--
	#### Dual Camera CAM-2MS2MF Compile
	##### HDMI+CAM-2MS2MF
	* modify dts
	```
	--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
	+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
	@@ -9,8 +9,8 @@
	 #include "rk3588s-firefly-aio-3588sg.dtsi"
	 //#include "rk3588s-firefly-aio-3588sg-ext.dtsi"

	-#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
	-//#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
	+//#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
	+#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
	```

	* Compile
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```

	#### HDMI TO MIPI_CSI(RK628D) Compile
	* modify dts
	```
	--- a/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
	+++ b/kernel-6.1/arch/arm64/boot/dts/rockchip/rk3588s-firefly-aio-3588sg.dts
	@@ -9,9 +9,9 @@
	 #include "rk3588s-firefly-aio-3588sg.dtsi"
	 //#include "rk3588s-firefly-aio-3588sg-ext.dtsi"

	-#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
	+//#include "rk3588s-firefly-aio-3588sg-cam-8ms1m.dtsi"
	 //#include "rk3588s-firefly-aio-3588sg-cam-2ms2mf.dtsi"
	-//#include "rk3588s-firefly-aio-3588sg-tf-hdmi-mipi-rk628.dtsi"
	+#include "rk3588s-firefly-aio-3588sg-tf-hdmi-mipi-rk628.dtsi"
	```

	* Compile
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
	-->
  </CodeBlockTab>

  <CodeBlockTab value="partial">
	### Step by step to compile [step]

	1. Compile the kernel:

	```
	cd ~/proj/RK3588_Android14.0/kernel-6.1
	export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64 firefly_defconfig android-14.config pcie.config
	msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8
	```

	2. Compile uboot:

	```
	cd ~/proj/RK3588_Android14.0/u-boot/
	./make.sh rk3588
	```

	3. Compile Android：

	```
	cd ~/proj/RK3588_Android14.0/
	source build/envsetup.sh
	lunch rk3588s_firefly_aio_3588sg-userdebug
	make installclean
	make -j8
	./mkimage.sh
	```

	### Packaged into unified firmware update.img [step]

	After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
	After packaging, it will be in rockdev/Image-rk3588s_firefly_aio_3588sg/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

	It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
	date.img and store it in rockdev \ Image directory.
### Build outputs [step]

	When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev/Image-rk3588s_firefly_aio_3588sg/. The following lists the image files used by general firmware:

	| Image | Instruction |
	|-------|-------------|
	| boot.img | including ramdis、kernel、dtb |
	| dtbo.img | Device Tree Overlays |
	| MiniLoaderAll.bin | including first level loader |
	| misc.img | including recovery-wipe boot symbol information, after flashing it will enter recovery |
	| parameter.txt | including partition information |
	| recovery.img | including recovery-ramdis、kernel、dtb |
	| super.img | including the contents of odm、vendor、system partitions |
	| uboot.img | including uboot image |
	| vbmeta.img | including avb verification information, used for AVB verification |
	| update.img | including the above img files to be flashed, can be used for the tool to directly flash the whole image package |

	For details about how to upgrade a partition image file, see [USB Firmware Upgrade](../../firmware_upgrade/upgrade_firmware.md).
  </CodeBlockTab>

  <CodeBlockTab value="ota">
	## OTA Compilation 

	### Firefly full OTA compilation and upgrade
	#### Add the -o parameter for OTA full compilation
	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug -o
	```

	#### Generate the full package path
	```
	out/target/product/rk3588s_firefly_aio_3588sg/rk3588s_firefly_aio_3588sg-ota-eng.xxx.zip 
	```
	#### Generate the base material package path
	```
	out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip
	```
	Copy the base material package and rename it, mainly for subcontracting upgrades.
	```
	cp out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip ~/ota/v1.zip 
	```

	#### Package compiled firmware and upgrades
	```
	./FFTools/mkupdate/mkupdate.sh -l rk3588s_firefly_aio_3588sg-userdebug
	```
	In the `rockdev/Image-rk3588s_firefly_aio_3588sg/` directory, find the corresponding firmware and burn it to the machine, then the machine can carry out the full package upgrade and the difference package upgrade,put 'update.zip' into /sdcard/ and restart the full package as prompted.

	### Generate the incremental OTA package
	#### Generate a new base material package 
	The v1.zip basic material package has been made, and another v2.zip material package needs to be made to generate the difference package. You need to modify the kernel DTS code or Android code, and then run the command again.

	```
	./FFTools/make.sh -d rk3588s-firefly-aio-3588sg -j8 -l rk3588s_firefly_aio_3588sg-userdebug -o
	```

	Regenerate a new base material package that we can copy.
	```
	 cp out/target/product/rk3588s_firefly_aio_3588sg/obj/PACKAGING/target_files_intermediates/rk3588s_firefly_aio_3588sg-target_files-eng.xxx.zip  ~/ota/v2.zip
	```
	#### Generate the incremental OTA package command
	```
	out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
	```
	#### Upgrade OTA incremental OTA package
	Put update.zip to the machine sdcard directory reboot, it will automatically recognize, and then click the steps to complete the upgrade.

	### OTA server setup

	Refer to [OTA Documentation](https://community.t-firefly.com/en/doc/download/164) to setup the sever.

</CodeBlockTab>
</CodeBlockTabs>

<Callout title="Some Introduction about Compiling" type="info">
	**Android14.0 can’t be upgraded directly kernel.img and resource.img**

	`kernel.img` and `resource.img` for Android14.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev/Image-rk3588s_firefly_aio_3588sg/ directory.

	**Compiling kernel generation separately boot.img**

	Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

	```
	cd ~/proj/RK3588_Android14.0/kernel-6.1
	export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
	msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3588s_firefly_aio_3588sg/boot.img rk3588s-firefly-aio-3588sg.img -j8
	```

	After compiling, `boot.img` in the kernel directory can be directly upgrade.
</Callout>

  </CodeBlockTab>

</CodeBlockTabs>
