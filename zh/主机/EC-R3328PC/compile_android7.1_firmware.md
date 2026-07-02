**下载编译须知：**
目前该Android7.1 版本不做主要维护，请用户编译选择Android7.1 industry 版本<font color="#dd0000">(主要维护)</font>，该版本在工业和平板和盒子等领域的使用>上范围更加广泛，而且性能稳定大批量生产验证过，该版本也作为我司主要维护版本，适用于我司RK3399系统的所有机型。

# 编译 Android7.1 固件


## 下载 Android7.1 SDK

**Android SDK 源码包比较大,可以通过如下方式获取 Android7.1 源码包：**[Android7.1 源码包]

下载完成后先验证一下 MD5 码：

```
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.001
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.002
$md5sum ~/firefly_rk3399_android7.1_git_20211220.7z.003

90145747b33ad6834cd065b594f6687f  firefly_rk3399_android7.1_git_20211220.7z.001
6f28a1d66b1d168a89bfd3a7923775f8  firefly_rk3399_android7.1_git_20211220.7z.002
dc4d87ba8574900eed84f60f9cbe3646  firefly_rk3399_android7.1_git_20211220.7z.003
```

确认无误后，就可以解压：

```
mkdir -p ~/proj/EC-R3328PC
cd ~/proj/EC-R3328PC
7z x ~/firefly_rk3399_android7.1_git_20211220.7z.001 -r -o.
git reset --hard
```

* 更新

注意：解压后务必要先更新下远程仓库。以下为从 gitlab 处更新的方法：
```
#1. 进入SDK根目录
cd ~/proj/EC-R3328PC

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399-nougat-bundle.git .bundle

#3. 若下载仓库失败，则可以从下方百度云下载[bundle压缩包]并解压到SDK根目录，解压指令如下：
7z x rk3399-nougat-bundle-20190612.7z -r -o. && mv rk3399-nougat-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```

百度云下载[[bundle压缩包]](http://www.t-firefly.com/doc/download/page/id/3.html#other18)



## EC-R3328PC 产品编译方法


[下载链接]: https://www.t-firefly.com/doc/download/84.html
[Android7.1 源码包]: https://www.t-firefly.com/doc/download/84.html#other_18
