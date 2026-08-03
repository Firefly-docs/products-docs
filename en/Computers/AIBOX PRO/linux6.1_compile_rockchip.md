# Compile Main Module Firmware
## Download SDK

Depand on **Main Module**, Please contact sales@t-firefly.com to get **RK3588 Kernel6.1 SDK** download link and read the **readme** file.

<font color=red>

**Notice:**
<br>
**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**<br>
**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**<br>
**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**<br>
**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**

</font>

* RK3588 SDK
    * At least update to `rk3588/linux6.1_release_v1.3.0e`

## Compile Debian Firmware
<font color=red> Download SDK First. </font>

!INCLUDE "./linux6.1_compile_debian_aibox-pro-3588.md"

## Compile Ubuntu Firmware
<font color=red> Download SDK First. </font>

!INCLUDE "./linux6.1_compile_ubuntu_aibox-pro-3588.md"

## Export Main Module Rootfs
Reference [Export device rootfs](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#export-device-system)

