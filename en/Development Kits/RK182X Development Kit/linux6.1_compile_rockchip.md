# Compile Main Module Firmware
## Download SDK

Depand on **Main Module**, Please contact sales@t-firefly.com to get **RK3588 Kernel6.1 SDK** or **RK3576 Kernel6.1 SDK** download link and read the **readme** file.

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
* RK3576 SDK
    * At least update to `rk3588/rk3576/linux_release_v1.3.0a`

## Compile Debian Firmware
<font color=red> Download SDK First. </font>

### Rootfs
* Download rootfs here [Debian rootfs(64-bit) Kernel6.1](https://community.t-firefly.com/en/doc/download/140), please use rootfs under kernel-6.1 folder.
* Decompress rootfs and create a symbolic link

#### RK3588 
```
# Decompress
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3588_debian_rootfs.img
cd ..
```

#### RK3576 
```
# Decompress
7z x debian12_xxxx_rootfs_xxxx.7z

# Move the rootfs image to sdk and then create a symbolic link.
mkdir ./SDK/prebuilt_rootfs/
mv debian12_xxxx_rootfs_xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf debian12_xxxx_rtoofs_xxxx.img rk3576_debian_rootfs.img
cd ..
```

### Config
#### RK3588
##### Core-3588JD4

```
./build.sh firefly_rk3588_aio-gs1n2-3588jd4-rk182x_debian_defconfig
```

##### Core-3588SJD4 AI

```
./build.sh firefly_rk3588_aio-gs1n2-3588sjd4-ai-rk182x_debian_defconfig
```

#### RK3576
##### Core-3576JD4

```
./build.sh firefly_rk3576_aio-gs1n2-3576jd4-rk182x_debian_defconfig
```

### Build
```
./build.sh all
```

The generated firmware at `output/update/` , eg: `AIO-GS1N2-3588JD4-RK182X_Debian.XXX.img`

## Compile Ubuntu Firmware
<font color=red> Download SDK First. </font>

### Rootfs
* Download rootfs here [Ubuntu rootfs(64-bit) Kernel6.1](https://community.t-firefly.com/en/doc/download/140), please use rootfs under kernel-6.1 folder.
* Decompress rootfs and create a symbolic link

#### RK3588 
```
# Decompress
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3588_ubuntu_rootfs.img
cd ..
```

#### RK3576
```
# Decompress
7z x Ubuntu22.04-xxxx.7z

mkdir ./SDK/prebuilt_rootfs/
mv Ubuntu22.04-xxxx.img ./SDK/prebuilt_rootfs/
cd ./SDK/prebuilt_rootfs/
ln -sf Ubuntu22.04-xxxx.img rk3576_ubuntu_rootfs.img
cd ..
```

### Config
#### RK3588
##### Core-3588JD4

```
./build.sh firefly_rk3588_aio-gs1n2-3588jd4-rk182x_ubuntu_defconfig
```

##### Core-3588SJD4 AI

```
./build.sh firefly_rk3588_aio-gs1n2-3588sjd4-ai-rk182x_ubuntu_defconfig
```

#### RK3576
##### Core-3576JD4

```
./build.sh firefly_rk3576_aio-gs1n2-3576jd4-rk182x_ubuntu_defconfig
```

### Build
```
./build.sh all
```

The generated firmware at `output/update/` , eg: `AIO-GS1N2-3588JD4-RK182X_Ubuntu.XXX.img`

## Export Main Module Rootfs
Reference [Export device rootfs](/docs/tools/development-tool/Rootfs-Export-Tool/ff-export-rootfs)



