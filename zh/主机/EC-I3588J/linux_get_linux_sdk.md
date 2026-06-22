### 下载 Firefly_Linux_SDK 分卷压缩包

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包 SDK。用户可以通过如下方式获取 Firefly_Linux_SDK 源码包：**[Firefly_Linux_SDK源码包]()

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split*
b633414d69240faa0c3bd755c255ede8  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file0
5cb1e2b63bbb4e3595c731038f6723fd  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file1
e06c0f29fd5a870e0942139a1877a470  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file2
4e14fbf72ccbb9b87f81d83a256205b0  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file3
be8b003703b51e7220e52bb36439a357  rk356x_linux_release_v1.2.0_20211019_split_dir/rk356x_linux_release_v1.2.0_20211019_firefly_split.file4
```

确认无误后，就可以解压：

<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

```bash
mkdir ~/proj/
cd ~/proj/
cat path/to/rk356x_linux_release_v1.2.0_20211019_split_dir/*firefly_split* | tar -xzv
```

### 同步代码

执行如下命令同步代码：

```bash
# 进入 SDK 根目录
cd ~/proj/rk356x_linux_release_20211019/

# 同步
.repo/repo/repo sync -l
.repo/repo/repo sync -c --no-tags
.repo/repo/repo start firefly --all
```

后续可以使用以下命令更新 SDK：

```bash
.repo/repo/repo sync -c --no-tags
```

因为网络环境等原因，`.repo/repo/repo sync -c --no-tags` 命令更新代码可能会失败，可多次反复执行。