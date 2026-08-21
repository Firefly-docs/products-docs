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

### rootfs
* Download root filesystem: [Debian Rootfs(64-bit) Kernel6.1](https://community.t-firefly.com/en/doc/download/420), please use the filesystem from the `kernel-6.1` directory.
* Extract rootfs and link it

#### RK3588
```
# Extract
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the extracted rootfs image to SDK and create a symbolic link
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

### Configure
#### Core-3588JD4

```
./build.sh firefly_rk3588_aibox-pro-3588jd4_rk182x_debian_defconfig
```

#### Core-3576JD4

```
./build.sh firefly_rk3576_aibox-pro-3576jd4-rk182x-m2_debian_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-3588JD4_Debian.XXX.img`
## Compile Ubuntu Firmware
<font color=red> Download SDK First. </font>

### rootfs
* Download root filesystem: [Ubuntu Rootfs(64-bit) Kernel6.1](https://community.t-firefly.com/en/doc/download/420), please use the filesystem from the `kernel-6.1` directory.
* Extract rootfs and link it

#### RK3588
```
# Extract
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

### Configure
### RK3588

```
./build.sh firefly_rk3588_aibox-pro-3588jd4_rk182x_ubuntu_defconfig
```

### RK3576

```
./build.sh firefly_rk3576_aibox-pro-3576jd4-rk182x-m2_ubuntu_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-3588JD4_Ubuntu.XXX.img`
## Export Main Module Rootfs
Reference [Export device rootfs](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#export-device-system)

