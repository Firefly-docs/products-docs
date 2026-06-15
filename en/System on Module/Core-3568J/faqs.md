# FAQS

## Build kernel, show "IO-Domain Checklist" Dialog
kernel new dts file `arch/arm64/boot/dts/rockchip/rk3568-firefly-aioj-DEMO.dts`and when build kernel, will show dialog
![](../../img/faq_rk356x_io-domain_checklist.png)

**Perform the following operations first**    
```
cp arch/arm64/boot/dts/rockchip/.rk3568-firefly-aioj.dtb.dts.tmp.domain arch/arm64/boot/dts/rockchip/.rk3568-firefly-aioj-DEMO.dtb.dts.tmp.domain
```

<font color=red>**Do not change the IO-Domain. Otherwise, IO may be damaged in the worst case.**</font>

  

## Changes in kernel/buildroot configuration doesn't take effect after compiling
After changing configuration through `make menuconfig`, compiling with `build.sh` and burning into device, the changes doesn't take effect. Then you check the configuration and find out that the previous changes have disappeared.

That's because the changes are only saved in temporary file `.config`. While compiling, `build.sh` will overwrite `.config` with configuration file.

So changes need to be saved in configuration file:
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
Then use `build.sh` to compile.

## How to adjust HDMI output resolution manually ?

In addition to adjusting the resolution in `Setting->Display->HDMI->Resolution`, we could also use command below to switch 4K resoultion and flash screen:

```
# 4k60:
setprop persist.sys.resolution.main 3840x2160@60 
# Set sys.display.timeline base on the default num to plus one for updating resolution
setprop sys.display.timeline 1
```
## The 3.5mm headphone jack is connected to the national standard headphone abnormal?

At present, Core-3568J only supports the American standard type of headphones(CTIA). For the national standard type of headphones(OMTP), the hardware is not compatible, and there will be the phenomenon of dual sound channels in both the left and right channels.

## What should I do if the boot is abnormal and restarts cyclically?

It may be that the power supply current is not enough. Please use a power supply with a voltage of 12V and a current of 2.5A~3A.

## What is the default username and password for Ubuntu?

* Username: `firefly`
* Password: `firefly`
* Switch super user `sudo -s`


## Write number tool to write SN, MAC address

<font color=red>**Note:**</font> If the eMMC erase operation is performed on the development board, the previously written data will also be cleared.

### Windows way
* Install RKDevInfoWriteTool
    * [Download link](http://en.t-firefly.com/doc/download/89.html#other_297)
* Select "RPMB" in **Settings** of RKDevInfoWriteTool
* Configure "SN", "WIFI MAC", "LAN MAC", "BT MAC", etc. in the **Settings** of RKDevInfoWriteTool as needed
* The development board enters loader mode
* RKDevInfoWriteTool performs **write** or **read** operations

For specific operations, please refer to the PDF document "RKDevInfoWriteTool User Guide" under the RKDevInfoWriteTool installation directory.

### Linux way

How to write the number of the development board itself

* Buildroot enable `BR2_PACKAGE_VENDOR_STORAGE`
* Read and write operations through the vendor_storage command
    * [Download link](http://en.t-firefly.com/doc/download/89.html#other_297)
     * SN
     ```shell
     vendor_storage -w VENDOR_SN_ID -t string -i cad895bedb8ee15f
     vendor_storage -r VENDOR_SN_ID -t hex -i /dev/null
     ```

     * LAN MAC
     ```shell
     vendor_storage -w VENDOR_LAN_MAC_ID -t string -i AABBCCDDEEFF
     vendor_storage -r VENDOR_LAN_MAC_ID -t hex -i /dev/null
     ```

        Others can be operated according to the prompt of `vendor_storage -h`.

For how to read the application, please refer to the `vendor_storage_read` function in `buildroot/package/rockchip/vendor_storage/vendor_storage.c`.

## On Ubuntu system, if there is no sound after plugging in headphones, what should I do?

`Menu` -> `Multimedia` -> `PulseAudio Volume Control` -> `Configuration` -> Select the sound card that is working and turn off the other sound card.

## How to make the system crawl LOG under Android?

1、`Settings (settings)` -> `About tablet (about tablet)` -> Click 7 times `Build number (version number)` 

2、`Settings (settings)` ->`System(system)`->`Advanced(advanced)`-> `Developer options (Developer options)` -> `Android LogSave`. After the function is turned on, the log will be generated under the root directory of the system `/data/vendor/logs`, which includes the system logcat and kernel kmsg.

## Problems encountered in compiling boot.img of Android
The system will boot in recovery mode without using `BOOT_IMG` to compile kernel, specify a boot.img like `BOOT_IMG=../rockdev/Image-rk3568_firefly_aioj/boot.img` when compiling kernel, the boot.img will update in directory `SDK/kernel` after compilation. 
```
make ARCH=arm64 BOOT_IMG=../rockdev/Image-rk3568_firefly_aioj/boot.img rk3568-firefly-aioj.img -j8
```
If `rockdev` is not available, please compile the SDK completely first, and refer to *Public Compile*.  

