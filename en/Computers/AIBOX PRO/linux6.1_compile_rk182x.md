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
For example, the SDK archive is `RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz`. (The specific version is subject to the latest release name on the cloud drive.)

```
mkdir rk182x_sdk
cd rk182x_sdk
tar xf RK182X_AI_COPROCESSOR_SDK_ALPHA_V1.X.X.tgz
.repo/repo/repo sync -l
```

### Bundle Update
Subsequent updates after version 1.1.0a will be released as bundles to reduce download time.
After downloading and decompressing the SDK base package above and completing synchronization, place the bundle package in the SDK root directory:

```
rk182x_sdk/
├── .repo/
└── bundle_xx_to_xx.tgz
```

Decompress the corresponding bundle package:

```
tar xzf bundle_xx_to_xx.tgz
```

Run the bundle's internal script from the SDK root directory:

```
./bundle_xx_to_xx/bundle_update.sh
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

Select `4`

```
Select Security Boot Mode:
1) Disable Secure Boot
2) Enable Secure Boot
3) Cancel
#? 
```
Select `1`

> This option is used to apply security encryption to the firmware. Incorrect operation may prevent the firmware from being upgraded later, so please choose carefully. For more information, refer to 1828SDK_Path/docs/Develop/Rockchip_RK1820_RK1828_User_Guide_SecureBoot_EN.pdf.


## Build
```
./build.sh
```

The generated software installation package is located at `output/firmware/rknn3_rk182x_m2_installer_arm64.tgz`

## Install
You need to manually install the RK1820/RK1828 software package, follow the steps below:
* Copy `rknn3_rk182x_m2_installer_arm64.tgz` to the main controller (RK3588 or RK3576)
* Decompress: `tar xzf rknn3_rk182x_m2_installer_arm64.tgz`
* Install: `./install.sh`
    * After installation and rebooting, the RK3588 or RK3576 system will automatically download the RK182X firmware and start the background service program upon startup.


## Others
### Version V 1.1.0
```
sudo rknn-smi -v
rknn-smi version              : 1.3.0
PCIe driver version           : 3.3.1
RC chips connect version      : 3.3.2
EP chips connect version      : 0.0.2
PCIe Device 0 firmware version: 1.1.0
rknn3 API version             : 1.1.0

```

## FAQ

### Currently Supported Models
For currently supported models and detailed information, refer to `SDK_Path/rknn/rknn3-runtime/doc/EN/00_RKNN3_SDK_Release_Notes_V1.1.0.pdf` in the SDK.

### rknn3 API Version shows NA
Install binutils; some rootfs may not have the strings command.

```sh
sudo apt update
sudo apt install binutils
```