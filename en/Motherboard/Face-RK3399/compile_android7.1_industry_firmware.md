# Compile Android7.1 Industry firmware


## Download compilation instructions:
Android7.1 industry version it is more widely used in industry and tablet and box fields, and has been verified for stable performance in mass production.
This version also serves as our main maintenance version and is applicable to all models of our RK3399 system.


## Compile Android7.1 industry firmware

### Download Android SDK

**The Android SDK source package is relatively large, you can go to the download page to get the Android7.1 source package:**
[Android7.1 industry SDK]

After downloading, verify the MD5 code:
```
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.001
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.002
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.003
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.004

3387ebaa3d8e43dd1164527d4054621b  firefly_rk3399_industry7.1_git_20211216.7z.001
a74bb71622add3d2b558dc5a0bfb51e4  firefly_rk3399_industry7.1_git_20211216.7z.002
00f3202d42559e02cefc5300e48e1690  firefly_rk3399_industry7.1_git_20211216.7z.003
fb64756ff7e24d9bb76ecd4678afb55f  firefly_rk3399_industry7.1_git_20211216.7z.004
```
After confirming that it is correct, you can unzip:
```
mkdir -p ~/proj/firefly-rk3399-Industry
cd ~/proj/firefly-rk3399-Industry
7z x ~/firefly_rk3399_industry7.1_git_20211216.7z.001 -r -o.
git reset --hard
```
Note: Be sure to update the remote warehouse after decompression.
The following is how to update from gitlab:
```
1. Enter the SDK root directory
cd ~/proj/firefly-rk3399-Industry

2. Download remote bundle repository
git clone https://gitlab.com/TeeFirefly/rk3399-industry-nougat-bundle.git .bundle

3. If the download warehouse fails, the current bundle warehouse is about 1.4G, so there may be stuck or failed problems during synchronization. You can download and unzip it from the Baidu cloud link below to the SDK root directory.

7z x rk3399-industry-nougat-bundle.7z  -r -o. && mv rk3399-industry-nougat-bundle/ .bundle/

4. Update the SDK, and subsequent updates do not need to pull the remote warehouse again, just execute the following command
.bundle/update

5. Follow the prompts to update the content to FETCH_HEAD, synchronize FETCH_HEAD to the firefly branch
git rebase FETCH_HEAD
```

Google Drive[[bundle download]](https://community.t-firefly.com/en/doc/download/3#other_230)




## Face-RK3399 industry version product compilation method

## Face-RK3399 Compiling method
### Overall Compilation

Face-RK3399 default display interface is MIPI DSI, at the time of compilation by adding `-o face` option to specify whether to compile the boot of Face recognition APK (FaceApp).  


* Without FaceApp

```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh  -d rk3399-firefly-face-mipi8 -j8 -l rk3399_firefly_face-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_face-userdebug
```
  
* With FaceApp  

```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh  -i face -d rk3399-firefly-face-mipi8 -j8 -l rk3399_firefly_face-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_face-userdebug
```

### Compile step by step

Before compilation, execute the following command to configure the environment variables:

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Compile the kernel:

```
cd ~/proj/firefly-rk3399/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly-face-mipi8.img
```

* Compile the uboot:

```
cd ~/proj/firefly-rk3399/u-boot/
make rk3399_box_defconfig
make ARCHV=aarch64 -j8
```

* Compile the Android (Without FaceApp):

```
cd ~/proj/firefly-rk3399/
source build/envsetup.sh
export FIREFLY_FACE_APP=false
lunch rk3399_firefly_face-userdebug
make installclean
make -j8
./mkimage.sh

```  

* Compile the Android (With FaceApp):

```
cd ~/proj/firefly-rk3399/
source build/envsetup.sh
export FIREFLY_FACE_APP=true
lunch rk3399_firefly_face-userdebug
make installclean
make -j8
./mkimage.sh

``` 

## Package into unified firmware - update.img

After compiling, you can package the unified firmware with the official script of Firefly, and execute the following command:

```
./FFTools/mkupdate/mkupdate.sh
```

When the packaging is complete, the unified firmware `Face-RK3399_Android1.1.2_DEFAULT_xxxxxx.img` will be generated under `rockdev/Image-rk3399_firefly_face/`.

Is also easy to package `update.img` under Windows. Copy the compiled files to `rockdev\Image` directory of AndroidTool, and then run `./mkupdate.bat` under rockdev directory to create `update.img` and store it in `rockdev\Image` directory.

## Upgrade partition images

`boot.img` and `system.img` will be repackaged when you execute `./mkimage.sh` during compilation. And the other related image files will be copy into `rockdev/Image-rk3399_firefly_face`.

The following is a list of image files commonly used:

* boot.img: Android's initial file image, which initializes and loads the system partition.
* kernel.img: The kernel image
* misc.img: misc partition image, responsible for initiating mode switching and parameter passing of first-aid mode.
* parameter.txt: Partition information for emmc.
* recovery.img: Image of first aid mode.
* resource.img: resource image, which contains boot image and kernel device tree information.
* system.img: Android system partition image, ext4 file system format.
* trust.img: sleep to wake up relevant files.
* rk3399_loader_v1.08.106.bin: Loader file.
* uboot.img: uboot file.

Refer to [How to upgrade firmware](upgrade_firmware.md) to burn the partition image file.

If you are using Windows system, copy the above image files to `rockdev\Image` directory of AndroidTool (The firmware upgrade tool under Windows), and then upgrade the partition image according to the upgrade document. The advantage is that you can use the default configuration without changing the path of the file.

`update.img` is convenient for firmware release, for end users to upgrade the system. It is generally convenient to use partition images during development.
## Other Android versions
* <font color=#ff0000 size=3>Main maintenance:</font>

   ["Compile Android7.1 Industry firmware"](compile_android7.1_industry_firmware.md)  



[Android7.1 industry SDK]: https://community.t-firefly.com/en/doc/download/3#other_230
