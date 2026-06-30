# 编译 Android7.1 Industry 固件

## 下载编译须知
Android7.1 industry 版本在工业和平板和盒子等领域的使用上范围更加广泛，而且性能稳定大批量生产验证过，
该版本也作为我司主要维护版本，适用于我司RK3399系统的所有机型。


## 下载 Android SDK

**Android SDK 源码包比较大,可以去下载页面来获取`Android7.1 industry`源码包：**
[Android7.1 industry源码包]

下载完成后先验证一下 MD5 码：
```
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.001
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.002
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.003
$md5sum ~/firefly_rk3399_industry7.1_git_20211216.7z.004

3387ebaa3d8e43dd1164527d4054621b  firefly_rk3399_industry7.1_git_20211216.7z.001
a74bb71622add3d2b558dc5a0bfb51e4  firefly_rk3399_industry7.1_git_20211216.7z.002
00f3202d42559e02cefc5300e48e1690  firefly_rk3399_industry7.1_git_20211216.7z.003
fb64756ff7e24d9bb76ecd4678afb55f  firefly_rk3399_industry7.1_git_20211216.7z.004
```
确认无误后，就可以解压：
```
mkdir -p ~/proj/firefly-rk3399-Industry
cd ~/proj/firefly-rk3399-Industry
7z x ~/firefly_rk3399_industry7.1_git_20211216.7z.001 -r -o.
git reset --hard
```
注意：解压后务必要先更新下远程仓库。
以下为从 gitlab 处更新的方法：
```
1. 进入SDK根目录
cd ~/proj/firefly-rk3399-Industry

2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399-industry-nougat-bundle.git .bundle

3. 若下载仓库失败，目前bundle仓库大约1.4G左右，所以同步的时候可能会出现卡住或失败的问题，可以从下方百度云链接下载并解压到SDK根目录，解压指令如下：

7z x rk3399-industry-nougat-bundle.7z  -r -o. && mv rk3399-industry-nougat-bundle/ .bundle/

4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可

.bundle/update

5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```

百度云下载[[bundle压缩包]](http://www.t-firefly.com/doc/download/page/id/85.html#other_369)


## 编译 Android SDK



## 其他安卓版本
* <font color=#ff0000 size=3>主要维护：</font>

   [《编译 Android7.1 industry 固件》](compile_android7.1_industry_firmware.md)  
* 支持但不维护：

   [《编译 Android7.1 固件》](compile_android7.1_firmware.md)     [《编译 Android8.1 固件》](compile_android8.1_firmware.md)  
[Android7.1 industry源码包]: http://www.t-firefly.com/doc/download/page/id/53.html#other_369
