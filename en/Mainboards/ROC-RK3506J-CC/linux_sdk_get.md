## Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>

**Attention:**

**1. SDK uses cross-compile, so download SDK to your X86_64 PC, do not download SDK to arm64 board.**

**2. Build environment needs to be Ubuntu18.04 (real PC or docker container), other versions may cause build failure.**

**3. Do not put SDK under shared directory of VM or non-English path.**

**4. Please get/build SDK as a normal user, root privilege are neither allowed nor required (except installing sth. with apt)**
</font>

### Install Tools
```
sudo apt update
sudo apt install -y repo git python
```

### Init Code Repository

* Method One

Download via repo, you can choose to get full SDK or BSP:

```bash
mkdir ~/proj/rk3588_sdk/
cd ~/proj/rk3588_sdk/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3588_linux_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3588_linux_bsp_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Linux SDK](https://en.t-firefly.com/doc/download/309.html)

After downloading, verify the MD5 code:

```bash
$ md5sum rk3588_linux_release_20230114_v1.0.6c_0*
c3bcb3f92bd139f72551c89f75d39bfa  rk3588_linux_release_20230114_v1.0.6c_00
ebb658571a645d4af1e2b569709480b7  rk3588_linux_release_20230114_v1.0.6c_01
9761cc324e9f7133500b590c441b0307  rk3588_linux_release_20230114_v1.0.6c_02
7adc9fe2158d7681554dce1def238f49  rk3588_linux_release_20230114_v1.0.6c_03
3d9201e3849b8a523c05920bebe28b39  rk3588_linux_release_20230114_v1.0.6c_04
6faaee006fe60fc9be60a64a01506cb6  rk3588_linux_release_20230114_v1.0.6c_05
```

After confirming that it is correct, you can unzip:
```bash
mkdir -p ~/proj/rk3588_sdk
cd ~/proj/rk3588_sdk
cat path/to/rk3588_linux_release_20230114_v1.0.6c_0* | tar -xv

# export data
.repo/repo/repo sync -l
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3588_sdk

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```
