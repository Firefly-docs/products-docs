### Download Firefly_Linux_SDK

* Method One

Download via repo:

```bash
mkdir ~/proj/rk356x_linux_release_20211019/
cd ~/proj/rk356x_linux_release_20211019/
repo init --no-clone-bundle --repo-url https://gitlab.com/firefly-linux/git-repo.git -u https://gitlab.com/firefly-linux/manifests.git -b master -m rk356x_linux_release.xml
```

* Method Two

Download Firefly_Linux_SDK sub-volume compressed package: [Firefly_Linux_SDK Source]()

After downloading, verify the MD5 code:

```bash
$ md5sum rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split*
b633414d69240faa0c3bd755c255ede8  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file0
5cb1e2b63bbb4e3595c731038f6723fd  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file1
e06c0f29fd5a870e0942139a1877a470  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file2
4e14fbf72ccbb9b87f81d83a256205b0  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file3
be8b003703b51e7220e52bb36439a357  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file4
```

After confirming that it is correct, you can unzip:

<font color=red>**Attention: To avoid unnecessary errors, please do not unzip the SDK in shared folders, mounted folders or non-english directories.**</font>

```bash
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split* | tar -xzv
```

### Sync Code

Execute the following command to synchronize the code:

```bash
# Enter the SDK root directory
cd ~/proj/rk356x_linux_release_20211019/

# Sync
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

You can use the following command to update the SDK later:

```bash
.repo/repo/repo sync -c --no-tags
```