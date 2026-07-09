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
* Download rootfs here [Debian rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/140.html), please use rootfs under kernel-6.1 folder.
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
* Download rootfs here [Ubuntu rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/140.html), please use rootfs under kernel-6.1 folder.
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
Reference [Export device rootfs](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#export-device-system)

# Compile RK1820/RK1828 Installer
## Download SDK
Please contact `sales@t-firefly.com` to get **RK182X SDK** download link.

<font color=red>

**Notice:**
<br>
**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**<br>
**2. We suggest to use Ubuntu20.04 (real PC or docker) to build, other OS may cause building failure**<br>
**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**<br>
**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**

</font>

<br>
eg: SDK is `rk182x_linux_release_20260611_v1.0.4a.tgz`.

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf rk182x_linux_release_20260611_v1.0.4a.tgz
.repo/repo/repo sync -l
```

## Config
Use `./build.sh config` to config.

```
Select board type:
1) RK182X EVB1
2) RK182X SODIMM
3) RK182X SODIMM USB
4) RK182X M2
5) Cancel
#? 
```

Select `2`

## Build
```
./build.sh
```

The generated software installation package is located at `output/firmware/rknn3_rk182x_sodimm_installer_arm64.tgz`

## Install
You need to manually install the RK1820/RK1828 software package, follow the steps below:
* Copy `rknn3_rk182x_sodimm_installer_arm64.tgz` to RK3588 or RK3576
* Decompress: `tar xzf rknn3_rk182x_sodimm_installer_arm64.tgz`
* Install: `./install.sh`
    * After installation and rebooting, the RK3588 or RK3576 system will automatically download the RK182X firmware and start the background service program upon startup.


## Others
### Version V1.0.0
```
sudo rknn-smi -v
rknn-smi version : 1.1.0
PCIe driver version : 3.3.0
RC chips connect version : 3.2.0
EP chips connect version : 3.2.0
PCIe Device 0 firmware version: 1.0.0
rknn3 API version : 1.0.0
```
