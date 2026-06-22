# FAQS

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

At present, iCore-3562JQ only supports the American standard type of headphones(CTIA). For the national standard type of headphones(OMTP), the hardware is not compatible, and there will be the phenomenon of dual sound channels in both the left and right channels.

## What should I do if the boot is abnormal and restarts cyclically?

It may be that the power supply current is not enough. Please check if the adapter has enough power.

## What is the default username and password for Ubuntu?

* Username: `firefly`
* Password: `firefly`
* Switch super user `sudo -s`

## How to make the system crawl LOG under Android?

1、`Settings (settings)` -> `About tablet (about tablet)` -> Click 7 times `Build number (version number)` 

2、`Settings (settings)` ->`System(system)`->`Advanced(advanced)`-> `Developer options (Developer options)` -> `Android LogSave`. After the function is turned on, the log will be generated under the root directory of the system `/data/vendor/logs`, which includes the system logcat and kernel kmsg.