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