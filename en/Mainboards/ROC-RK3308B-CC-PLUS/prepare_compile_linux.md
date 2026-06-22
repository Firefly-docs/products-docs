## Build the build environment

This chapter introduces the compilation process of the Linux SDK

<font color=red>**Note: (1) It is recommended to develop in the Ubuntu 18.04 system environment. If other system versions are used, the compilation environment may need to be adjusted accordingly.** </font>

<font color=red>**(2) Compile with ordinary user, do not compile with root user authority.** </font>

### Install dependencies

```bash
sudo apt-get install repo git ssh make gcc libssl-dev liblz4-tool \
expect g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip ncurses-dev python-pyelftools
````

### Download Firefly_Linux_SDK

* Method 1 (recommended)

**Because the Firefly_Linux_SDK source code package is relatively large, some users' computers do not support files above 4G or the network transmission of a single file is slow, so we use the method of volume compression to package the SDK. Users can obtain the Firefly_Linux_SDK source package in the following ways: **[Firefly_Linux_SDK source package](https://www.t-firefly.com/doc/download/97.html#other_300)

After the download is complete, verify the MD5 code first:

```bash
$ md5sum rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split*
a97f218aed44be1b9a423a2868e09335  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file0
3f3936a72635b75613b89a4bf8d9de39  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file1
794ea504543a797d972c2112dfef2f3f  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file2
0de1858d73e2841790006e487ff70a19  rk3308_linux_release_v1.5.0a_20221212_split_dir/rk3308_linux_release_v1.5.0a_20221212_firefly_split.file3
````

After confirming that it is correct, you can unzip it:

<font color=red>**Note: Do not decompress the SDK in shared folders, mount folders and non-English directories to avoid unnecessary errors**</font>

```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3308_linux_release_v1.5.0a_20221212_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
````

* Method Two

Pulling the code through repo, this method has high network requirements and can be used under conditions

You can choose to get the full SDK or BSP:

```bash
mkdir ~/proj/rk3308_linux_release_v1.5.0a_20221212/
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_release.xml

## BSP (only the base repository and compilation tools are included)
## BSP includes device/rockchip , docs , kernel , u-boot , rkbin , tools and cross compile chain
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3308_linux_bsp_release.xml
````

### Sync code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3308_linux_release_v1.5.0a_20221212/

# Synchronize
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
````

The SDK can then be updated with the following command:

```bash
.repo/repo/repo sync -c --no-tags
````

Due to the network environment and other reasons, the `.repo/repo/repo sync -c --no-tags` command may fail to update the code, and it can be executed repeatedly.