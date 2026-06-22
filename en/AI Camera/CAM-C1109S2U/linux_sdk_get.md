## Download Firefly_Linux_SDK

First prepare an empty folder to place SDK, better under home, here we use `~/proj` as example.

<font color=red>

**Attention:**

**1. SDK uses cross-compile, so download SDK to your X86_64 PC, do not download SDK to arm64 board.**

**2. Build environment needs to be Ubuntu18.04 (real PC or docker container), other versions may cause build failure.**

**3. Do not put SDK under shared directory of VM or non-English path.**

**4. Please get/build SDK as a normal user, root privilege are neither allowed nor required (except installing sth. with apt)**
</font>


<font color=red>**Very important:**</font>

Before development, please update the SDK to the latest version according to the WIKI, especially for users who use the CS-R1 cluster server (if the code version of the CS-R1 user is too old and the software level configuration does not match the hardware, it may cause <font color=red>**hardware damage**</font>).

For `CORE-1126-JD4/CORE-1109-JD4` series, please download `rv1126_rv1109_linux_release` Linux SDK package.

If you want to develop UVC applications for the `CORE-1126-JD4/CORE-1109-JD4` series, please download the `rv1126_rv1109_linux_ai_camera_release` Linux SDK package.

### SDK Development Kit Description

`rv1126_rv1109_linux_release` SDK Description:
1. `CORE-1126-JD4/CORE-1109-JD4` series products can develop IPC firmware (IPC is the network camera)
2. `CAM-C1126S2U/CAM-C1109S2U` series products can develop Facial_gate firmware (Facial_gate is the face recognition gate)

`rv1126_rv1109_linux_ai_camera_release` SDK description:
1. `CORE-1126-JD4/CORE-1109-JD4` series products can develop AI_UVC firmware (UVC means driver-free camera)
2. `CAM-C1126S2U/CAM-C1109S2U` series products can develop AI_UVC firmware (UVC means driver-free camera)

Full SDK and BSP SDK Description:

`rv1126_rv1109_linux_release` and `rv1126_rv1109_linux_ai_camera_release` are both FULL SDKs.

`rv1126_rv1109_linux_bsp_release` and `rv1126_rv1109_linux_ai_camera_bsp_release` are both BSP SDKs.

The difference between the FULL SDK and the BSP SDK:
1. The FULL SDK can compile the buildroot system. The file size is relatively large. It contains the hardware acceleration file directory provided by rockchip. It is recommended to pull the FULL SDK for the first use.
2. The BSP SDK cannot compile the buildroot system. But the file size is relatively small. It is provided to customers who only need to update the partition firmware such as u-boot and kernel based on the public version of the firmware.

The similarities between the FULL SDK and the BSP SDK:
1. Both can compile and package Debian10 firmware.


### Install Tools
```
sudo apt update
sudo apt install -y repo git python
```

### Init Code Repository

* Method One

Download via repo, you can choose to get FULL SDK or BSP SDK:

* `rv1126_rv1109_linux_release` FULL SDK pull method
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```

* `rv1126_rv1109_linux_bsp_release` BSP SDK pull method
```bash
mkdir ~/proj/rv1126_sdk/
cd ~/proj/rv1126_sdk/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_bsp_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```



* `rv1126_rv1109_linux_ai_camera_release` FULL SDK pull method

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```



* `rv1126_rv1109_linux_ai_camera_bsp_release` BSP SDK pull method

```bash
mkdir ~/proj/rv1126_sdk_ai/
cd ~/proj/rv1126_sdk_ai/

repo init  --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git --no-repo-verify -u https://gitlab.com/firefly-linux/manifests.git -b master -m rv1126_rv1109_linux_ai_camera_bsp_release.xml

repo sync -c
# You need to repeatedly confirm whether the code download is successful
```

Note: The default repo tool uses python2 to pull code. If the Linux PC system is relatively new, the default version may be python3. If the code cannot be pulled, please manually set the python version of the Linux PC system to python2. And use the `.repo/repo/repo` tool in the current directory to pull the code. View the default python version of the Linux PC system:

```shell
$ ls -l /usr/bin/python
lrwxrwxrwx 1 root root 9 4月  16  2018 /usr/bin/python -> python2.7*
```



* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Linux SDK](https://en.t-firefly.com/doc/download/page/id/85.html)

Note: This method can only obtain the complete SDK. The BSP SDK needs to be obtained using method 1.

* `rv1126_rv1109_linux_release` SDK download

After downloading, verify the MD5 code:

```bash
md5sum rv1126_rv1109_linux_release_20211022.tgz
596c6bc6bb3095aea97d54c9df4cf333
```

After confirming that it is correct, you can unzip:
```bash
mkdir -p ~/proj/rv1126_sdk
cd ~/proj/rv1126_sdk
mv /path/to/rv1126_rv1109_linux_release_20211022.tgz ./
tar xvf rv1126_rv1109_linux_release_20211022.tgz

# This compressed package contains a .repo directory. After decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

# You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags

# Due to network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code. You can execute it repeatedly.
```

* `rv1126_rv1109_linux_ai_camera_release` SDK download

After downloading, verify the MD5 code:

```bash
md5sum rv1126_rv1109_linux_ai_camera_release_20210306.tgz
26fcb17ddd82a43d396eb3eac4b7479f
```

After confirming that it is correct, you can unzip:
```bash
mkdir -p ~/proj/rv1126_sdk_ai
cd ~/proj/rv1126_sdk_ai
mv /path/to/rv1126_rv1109_linux_ai_camera_release_20210306.tgz ./
tar xvf rv1126_rv1109_linux_ai_camera_release_20210306.tgz

# This compressed package contains a .repo directory. After decompression, perform the following operations in the current directory
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all

# You can use the following command to update the SDK later
.repo/repo/repo sync -c --no-tags

# Due to network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code. You can execute it repeatedly.
```

### Synchronize Code

Execute the following command to synchronize code:

* Enter the root directory of rv1126_rv1109_linux_release SDK

```bash
cd ~/proj/rv1126_sdk
```

* Enter the root directory of rv1126_rv1109_linux_ai_camera_release SDK

```bash
cd ~/proj/rv1126_sdk_ai
```

* Synchronize command

```bash
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```

Due to network environment and other reasons, `.repo/repo/repo sync -c --no-tags` The command to update the code may fail and can be executed repeatedly.

