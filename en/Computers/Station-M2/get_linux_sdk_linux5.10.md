
### Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>**Attention: To avoid unnecessary errors, please do not place/unzip the SDK in VM shared folders or non-english directories.**</font>

Get SDK needs:
```
sudo apt update
sudo apt install -y repo git python
```
* Method One

Download via repo, you can choose to get full SDK or BSP:

```bash
# Create SDK directory
mkdir ~/proj/rk356x_sdk-linux5.10
cd ~/proj/rk356x_sdk-linux5.10

## Full SDK
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_release.xml

## BSP (Only include some basic repositories and compile tools(Only for ubuntu & debian))
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://github.com/Firefly-rk-linux-utils/git-repo.git --no-repo-verify -u https://github.com/Firefly-rk-linux/manifests.git -b master -m rk356x_linux5.10_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [rk356x_linux5.10_release_20241220_v1.4.0c](), follow the README document:

```bash
└── rk356x_linux5.10_release_xxx
    ├── linux_sdk_tar
    │   ├── rk356x_linux5.10_release_xxx.sdk.split00
    │   ├── rk356x_linux5.10_release_xxx.sdk.split01
    │   ├── rk356x_linux5.10_release_xxx.sdk.split02
    │   ├── rk356x_linux5.10_release_xxx.sdk.split03
    ├── md5sum.txt
    ├── README_EN.txt
    ├── README_ZH.txt
    └── sdk_tools.sh
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_sdk-linux5.10

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```

### Directory

```bash
.
├── app
├── buildroot														# Buildroot root filesystem build directory
├── build.sh -> device/rockchip/common/build.sh					# Compile script
├── device															# Compile related configuration files
├── docs															# Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> buildroot/build/Makefile
├── prebuilts														# Cross compilation toolchain
├── rkbin
├── tools															# Tools directory
└── u-boot															# U-Boot
```