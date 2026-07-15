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
* Download root filesystem: [Debian Rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/333.html), please use the filesystem from the `kernel-6.1` directory.
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
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_debian_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-G2-3588JD4_Debian.XXX.img`

## Compile Ubuntu Firmware
<font color=red> Download SDK First. </font>

### rootfs
* Download root filesystem: [Ubuntu Rootfs(64-bit) Kernel6.1](https://en.t-firefly.com/doc/download/333.html), please use the filesystem from the `kernel-6.1` directory.
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
./build.sh firefly_rk3588_aibox-pro-g2-3588jd4_rk182x_ubuntu_defconfig
```

### Build
```
./build.sh all
```

The generated firmware is in the `output/update/` directory, e.g., `AIBOX-PRO-G2-3588JD4_Ubuntu.XXX.img`

## Export Main Module Rootfs
Reference [Export device rootfs](https://wiki.t-firefly.com/en/Firefly-Linux-Guide/first_use.html#export-device-system)

# Compile RK1820/RK1828 Installer
## Download SDK
Please contact `sales@t-firefly.com` to get **RK182X SDK** download link.

<font color=red>

**Notice:**
<br>
**1. SDK use cross-compilation, so use SDK in x86_64 PC, do not download SDK to the device**<br>
**2. We suggest to use Ubuntu20.04 or Ubuntu22.04 (real PC or docker) to build, other OS may cause building failure**<br>
**3. Do not place or decompress the SDK archive in Virtual Machine share folder or non-english folder**<br>
**4. Please use the regular user to get/compile the SDK, use root privilege may cause building failure**

</font>

<br>
eg: SDK is `RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.0.4.tgz`.

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.0.4.tgz
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
* Copy `rknn3_rk182x_sodimm_installer_arm64.tgz` to the main controller (RK3588 or RK3576)
* Decompress: `tar xzf rknn3_rk182x_sodimm_installer_arm64.tgz`
* Install: `./install.sh`
    * After installation and rebooting, the RK3588 or RK3576 system will automatically download the RK182X firmware and start the background service program upon startup.


## Others
### Version V1.0.4
```
sudo rknn-smi -v
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.0
RC chips connect version      : 3.3.1
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.0.4
rknn3 API version             : NA
```
