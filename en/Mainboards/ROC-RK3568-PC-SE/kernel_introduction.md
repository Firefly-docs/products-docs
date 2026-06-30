# Kernel

## Introduction

Firefly kernel does not come with all kernel features. If the default firmware does not meet your needs, such as lacking USB CAN support, users need to customize configuration and rebuild kernel.

## Customize

First need to get SDK, how to get SDK and prepare environment please refer to:

[Linux Development](prepare_compile_linux.md)

[Android Development](prepare_compile_android.md)

Then enable new features:

* Enter SDK/kernel, write configuration into .config :
```bash
# Linux
make ARCH=arm64 firefly_linux_defconfig

# Android
make ARCH=arm64 firefly_defconfig
```

* Enter configuration menu
```bash
make ARCH=arm64 menuconfig
```
![](../../../rk356x_img/kernel_menuconfig.jpg)

* Usage

1. Ahead option `*` means enabled and build-in, blank means disabled, `M` means enabled and build as module
1. `[]`only support build-in (Press Y) or disable (Press N)，besides Y and N `<>`can also be chose to build as module (Press M)；
1. Option ends with `—>` means there is sub-directory under it
1. Double-press `Esc` to exit, press `?` to show help, press `/` to global search
1. Up and down around the four direction keys is to move the cursor, `Enter` to select

* Notice

Select to build as module (M) requires follow-up installation, so select build-in (Y) to avoid installation step is recommended while adding few features.

Press the number at the left of an option in searching result page to directly jump to where the option locates.

![](../../../rk356x_img/kernel_menuconfig_search.jpg)

It is recommended to enable some options in same group. For example, all options in the picture below belong to the same function: CAN USB. 
So enable them all to get full CAN USB support.

![](../../../rk356x_img/kernel_menuconfig_demo.jpg)

Have doubts about an option, you can move cursor on it and press `?` to see help message.

* Save

After enabling needed options, move cursor to `Save` and press `Enter` for 3 times.

Then continually double press `Esc` to exit.

Back to terminal, save your modification into configuration file:
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig (Android is firefly_defconfig)
```

## Compile

Back to SDK root directory:
```bash
# Choose board mk file
./build.sh xxxx.mk
# Compile kernel
./build.sh kernel
```
The output file is `SDK/kernel/boot.img`

## Burning

Refer to [Upgrade Partition Image](03-upgrade_firmware.html#upgrade-partition-image)