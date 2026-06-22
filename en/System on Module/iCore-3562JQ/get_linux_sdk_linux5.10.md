
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
mkdir ~/proj/rk3562_sdk
cd ~/proj/rk3562_sdk

# Please contact sales@t-firefly.com to get repo urls.
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: Please contact sales@t-firefly.com to get download urls.

Extract the SDK:

```bash
# add execution permissions
cd /path/to/rk3562_release
chmod +x ./sdk_tools.sh

# create SDK directory
mkdir -p ~/proj/rk3562_sdk

# unzip
./sdk_tools.sh --unpack -C ~/proj/rk3562_sdk

# restore working directory
./sdk_tools.sh --sync -C ~/proj/rk3562_sdk
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3562_sdk

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
├── buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh     # build script
├── device/rockchip                                         # repository for build and config management
├── docs                                                    # useful documents
├── external                                                # some components
├── kernel
├── Makefile -> device/rockchip/common/Makefile
├── output                                                  # build output
├── prebuilt_rootfs                                         # to place prebuilt rootfs
├── prebuilts/gcc/linux-x86/                                # cross-build tool chain
├── README.md -> device/rockchip/common/README.md
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── rockdev -> output/firmware                              # build output of different partition images
├── tools                                                   # some tools, include upgrade_tool
└── u-boot
```