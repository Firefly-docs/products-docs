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
mkdir ~/proj/rk356x_linux_release_v1.3.0b_20221213/
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Firefly_Linux_SDK Source]()

After downloading, verify the MD5 code:

```bash
$ md5sum rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split*
409b81a9ed3bb9a7d6af91223836cad5  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file00
75cef82f2bf91052a7d3c6f0b8405a89  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file01
6f20f62e9652f8f999692587a2ac4b79  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file02
113acbbcd18d3abe0552ef296e983a3f  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file03
624c88a4da2eaa4a48f380783b126d00  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file04
1cf861afb0b36c9ebcf26a7d6effb260  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file05
3009e46fc14481e77fe7ec143e217de4  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file06
5e1cc90b99e34f20b75fb506d3e9bcd7  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file07
1a512fa7c9e2fd1a0781f8d40e228402  rk356x_linux_release_v1.3.0b_20221213_split_dir/rk356x_linux_release_v1.3.0b_20221213_firefly_split.file08
```

After confirming that it is correct, you can unzip:
```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.3.0b_20221213_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_linux_release_v1.3.0b_20221213/

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
├── debian															# Debian root filesystem compilation directory
├── device															# Compile related configuration files
├── docs															# Documentation
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel															# Kernel
├── Makefile -> buildroot/build/Makefile
├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh	# Link script
├── prebuilts														# Cross compilation toolchain
├── rkbin
├── rkflash.sh -> device/rockchip/common/rkflash.sh				# Flash script
├── tools															# Tools directory
└── u-boot															# U-Boot
```