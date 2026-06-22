
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
mkdir ~/proj/rk356x_amp_sdk
cd ~/proj/rk356x_amp_sdk

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, hal, rt-thread, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_amp_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [rk356x_amp_release_20240607_v0.0.1a]()

Extract the SDK:

```bash
# add execution permissions
cd /path/to/rk356x_amp_release_20240607_v0.0.1a
chmod +x ./sdk_tools.sh

# create SDK directory
mkdir -p ~/proj/rk356x_amp_sdk

# unzip
./sdk_tools.sh --unpack -C ~/proj/rk356x_amp_sdk

# restore working directory
./sdk_tools.sh --sync -C ~/proj/rk356x_amp_sdk
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_amp_sdk

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
├── buildroot   # Buildroot root filesystem build directory
├── build.sh    # Compile script
├── device      # Compile related configuration
├── docs        # SDK documentation
├── envsetup.sh
├── external
├── hal
│ ├── doc       # HAL documentation
│ ├── html      # HAL doxygen generates driver documentation
│ ├── lib       # HAL driver code
│ ├── project   # HAL project directory
│ ├── test      # HAL test code
│ └── tools     # HAL related tools
├── kernel      # Kernel
├── rt-thread   # RT-Thread
├── prebuilts   # SDK compilation toolchain (including compilation U-Boot, Kernel, HAL compilation toolchain)
├── rkbin
├── tools
│ ├── linux     # Linux platform burning tool
│ ├── mac       # Mac platform burning tool
│ └── windows   # Windows platform burning tools and drivers
└── u-boot      # U-boot
```