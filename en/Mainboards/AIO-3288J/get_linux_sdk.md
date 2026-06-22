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
mkdir ~/proj/rk3288_linux_release_v2.5.0a_20230510/
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

## Full SDK
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_release.xml

## BSP (Only include some basic repositories and compile tools)
## BSP includes device/rockchip, docs, kernel, u-boot, rkbin, tools and cross-compile toolchian
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk3288_linux_bsp_release.xml
```

* Method Two

Download the SDK source code package.

Download: [Firefly_Linux_SDK](http://en.t-firefly.com/doc/download/16.html#other_306)

After downloading, verify the MD5 code:

```bash
$ md5sum rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split*
f3b9309c574e04491a9e9f3e7a5c5540  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file0
fb0cd76f518405441bbc2ad2334ee6b9  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file1
cb26fddb36d49ff6b79f2fc934a731c1  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file2
43aa8447d4ef303ce41274c6ea7b00a2  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file3
c6c1ed77033af3d26be24974dda1b2aa  rk3288_linux_release_v2.5.0a_20230510_split_dir/rk3288_linux_release_v2.5.0a_20230510_firefly_split.file4
```

After confirming that it is correct, you can unzip:

```bash
# unzip
mkdir ~/proj/
cd ~/proj/
cat path/to/rk3288_linux_release_v2.5.0a_20230510_split_dir/*firefly_split* | tar -xzv

# export data
.repo/repo/repo sync -l
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk3288_linux_release_v2.5.0a_20230510/

# Sync
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```