# Compile Android firmware

<CodeBlockTabs defaultValue="a14">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="a14">Android 14</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="a14">

## Download Android14.0 SDK [step]

SDK source code and bundle compression package are stored in the Google Drive.

### Download Android SDK
* The SDK can be obtained by email. Send the order number to <font color=red>sales@t-firefly.com</font> and indicate the required SDK name [firefly_rk3576_android14.0_git_20241126]()

* After downloading, verify the MD5 code:

    ```
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.001
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.002
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.003
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.004
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.005
    $ md5sum /path/to/firefly_rk3576_android14.0_git_20241126.7z.006

    e368c4274125e754eeea35a97306e47b  firefly_rk3576_android14.0_git_20241126.7z.001
    d4436c73b800523dbb1df2ae0d524b26  firefly_rk3576_android14.0_git_20241126.7z.002
    9f6a7ed4f01a8952c135cc5bf6fd4ecd  firefly_rk3576_android14.0_git_20241126.7z.003
    3d2babe7c2c9f737d23e8843c2df3117  firefly_rk3576_android14.0_git_20241126.7z.004
    1d9ae98cc3e0959de19bc1dd628ed112  firefly_rk3576_android14.0_git_20241126.7z.005
    09e35581108741bf4a564829edcd1a5f  firefly_rk3576_android14.0_git_20241126.7z.006

    ```

* After confirming that it is correct, we can unzip:

    ```
    cd ~/proj/
    7z x ./firefly_rk3576_android14.0_git_20241126.7z.001 -oRK3576_Android14.0
    cd ./RK3576_Android14.0
    git reset --hard
    ```

   <font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

#### Second, Update SDK
How to update code after downloading SDK:
```
1. Enter the SDK root directory
cd ~/proj/RK3576_Android14.0

2. we can download and unzip it from the cloud disk link below to the SDK root directory.

7z x rk3576-android14.0-bundle.7z  -r -o. && mv rk3576-android14.0-bundle .bundle

3. Update the SDK
.bundle/update

4. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD

```

Google Driver [Android14.0 Bundle](https://community.t-firefly.com/en/doc/download/255)。

<!-- 2. Download remote bundle repository
git clone https://gitlab.com/T-Firefly/rk3576-android14.0-bundle.git .bundle

3. If the download warehouse fails, it may be stuck or failed problems during synchronization. 
we can download and unzip it from the cloud disk link below to the SDK root directory.
 -->


## Core-3576JD4 product compilation method [step]

<CodeBlockTabs defaultValue="full">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="full">Full compilation</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="partial">Step by step</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="ota">OTA Compilation</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="full">
### The overall compilation

#### HDMI Firmware Compilation
```
./FFTools/make.sh -j8 -l rk3576_firefly_aio_3576_jd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576_jd4-userdebug
```

#### 10.1' MIPI V3S Firmware Compilation：
```
./FFTools/make.sh -j8 -l 
./FFTools/mkupdate/mkupdate.sh -l 
```


  </CodeBlockTab>

  <CodeBlockTab value="partial">
### Step by step to compile

1. Compile the kernel:

```
cd ~/proj/RK3576_Android14.0/kernel-6.1
export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576_jd4/boot.img rk3576-firefly-aio-3576jd4.img -j8

```

2. Compile uboot:

```
cd ~/proj/RK3576_Android14.0/u-boot/
make rk3576_defconfig
./make.sh --spl-new
```

3. Compile Android：

```
cd ~/proj/RK3576_Android14.0/
source build/envsetup.sh
lunch rk3576_firefly_aio_3576_jd4-userdebug
make installclean
make -j8
./mkimage.sh
```

### Packaged into unified firmware update.img

After compilation, you can use Firefly official scripts to package into unified firmware, execute the following command：
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576_jd4-userdebug
```
After packaging, it will be in rockdev/Image-rk3576_firefly_aio_3576_jd4/ Generate unified firmware under the directory: (product name )XXX_XXX_date XXX.img

It is also very simple to package the unified firmware update.img under Windows. Copy the generated files to the rockdev \ Image directory of AndroidTool, and then run the mkupdate.bat batch file under the rockdev directory to create up
date.img and store it in rockdev \ Image directory.
  </CodeBlockTab>

  <CodeBlockTab value="ota">
## OTA Compilation 

### Firefly full OTA compilation and upgrade
#### Add the -o parameter for OTA full compilation
```
./FFTools/make.sh -d rk3576-firefly-aio-3576jd4 -j8 -l rk3576_firefly_aio_3576_jd4-userdebug -o
```

#### Generate the full package path
```
out/target/product/rk3576_firefly_aio_3576_jd4/rk3576_firefly_aio_3576_jd4-ota-eng.xxx.zip 
```
#### Generate the base material package path
```
out/target/product/rk3576_firefly_aio_3576_jd4/obj/PACKAGING/target_files_intermediates/rk3576_firefly_aio_3576_jd4-target_files-eng.xxx.zip
```
Copy the base material package and rename it, mainly for subcontracting upgrades.
```
cp out/target/product/rk3576_firefly_aio_3576_jd4/obj/PACKAGING/target_files_intermediates/rk3576_firefly_aio_3576_jd4-target_files-eng.xxx.zip ~/ota/v1.zip 
```

#### Package compiled firmware and upgrades
```
./FFTools/mkupdate/mkupdate.sh -l rk3576_firefly_aio_3576_jd4-userdebug
```
In the `rockdev/Image-rk3576_firefly_aio_3576_jd4/` directory, find the corresponding firmware and burn it to the machine, then the machine can carry out the full package upgrade and the difference package upgrade,put 'update.zip' into /sdcard/ and restart the full package as prompted.

### Generate the incremental OTA package
#### Generate a new base material package 
The v1.zip basic material package has been made, and another v2.zip material package needs to be made to generate the difference package. You need to modify the kernel DTS code or Android code, and then run the command again.

```
./FFTools/make.sh -d rk3576-firefly-aio-3576jd4 -j8 -l rk3576_firefly_aio_3576_jd4-userdebug -o
```

Regenerate a new base material package that we can copy.
```
 cp out/target/product/rk3576_firefly_aio_3576_jd4/obj/PACKAGING/target_files_intermediates/rk3576_firefly_aio_3576_jd4-target_files-eng.xxx.zip  ~/ota/v2.zip
```
#### Generate the incremental OTA package command
```
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```
#### Upgrade OTA incremental OTA package
Put update.zip to the machine sdcard directory reboot, it will automatically recognize, and then click the steps to complete the upgrade.

### OTA server setup

Refer to [OTA Documentation]() to setup the sever.
  </CodeBlockTab>
</CodeBlockTabs>

<Callout title="Some Introduction about Compiling" type="info">
	**Android14.0 can’t be upgraded directly kernel.img and resource.img**

	`kernel.img` and `resource.img` for Android14.0 are included in `boot.img`, After compiling the kernel, you need to run the `./mkimage.sh` command in the android root directory to repackage `boot.img`, and then upgrade `boot.img` of the rockdev/Image-rk3576_firefly_aio_3576_jd4/ directory.

	**Compiling kernel generation separately boot.img**

	Principle of compilation: in the kernel directory, the generated `kernel.img` And `resource.img` Replace with old `boot.img`, so you need to use BOOT_IMG=XXX parameter specification `boot.img` when compiling. The command is as follows:

	```
	cd ~/proj/RK3576_Android14.0/kernel-6.1
	export PATH=../prebuilts/clang/host/linux-x86/clang-r487747c/bin:$PATH
	alias msk='make CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 LLVM_IAS=1'
	msk ARCH=arm64 firefly_defconfig android-14.config rk3576.config
	msk ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3576_firefly_aio_3576_jd4/boot.img rk3576-firefly-aio-3576jd4.img -j8
	```

	After compiling, `boot.img` in the kernel directory can be directly upgrade.
</Callout>

## Partition image

When compiling, executing `./mkimage.sh` will repackage `boot.img` and `super.img`, and copy other related image files to the directory rockdev/Image-rk3576_firefly_aio_3576_jd4/. The following lists the image files used by general firmware:

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
</CodeBlockTabs>

## FAQs

### git clone remote bundle repository failed

Q: Error occurred in git clone remote bundle repository while [updating SDK](#second-update-sdk):

```
$ git clone https://gitlab.com/T-Firefly/rk3588-Android14.0-bundle.git .bundle
Cloning into '.bundle'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20), done.
remote: Compressing objects: 100% (18), done.
error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: unpack-objects failed
```

A: The buffer size is insufficient and needs to be enlarged:

```
git config --global https.postBuffer 1048576000
```