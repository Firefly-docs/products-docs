# 编译 Android8.1 固件

### 下载 Android SDK

由于 SDK 较大，可以取下载页面选择云盘下载 Firefly-RK3399_Android8.1_xxx.7za(可以直接使用Firefly-RK3399源码包)：
[Android8.1 源码包]

下载完成后，在解压前先校验下 MD5 码：
```
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.001
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.002
$md5sum ~/firefly_rk3399_android8.1_git_20211217.7z.003

48b4cfcb538cdc39ea53349989691617  firefly_rk3399_android8.1_git_20211217.7z.001
05776d4966d92efdcb88170355d8fbd4  firefly_rk3399_android8.1_git_20211217.7z.002
c83f54d89b4cde6eb620b92cd2e99a05  firefly_rk3399_android8.1_git_20211217.7z.003
```

然后解压：
```
mkdir -p ~/proj/roc-rk3399-pc
cd ~/proj/roc-rk3399-pc
7z x ~/firefly_rk3399_android8.1_git_20211217.7z.001 -r -o.
git reset --hard
```

更新远程仓库：
```
git remote rm gitlab
git remote add gitlab https://gitlab.com/TeeFirefly/firenow-oreo-rk3399.git
```

从 gitlab 处同步源码：
```
git pull gitlab firefly-rk3399:firefly-rk3399
```

也可以到如下地址在线查看源码：
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)


[Android8.1 源码包]: http://www.t-firefly.com/doc/download/65.html#other_144


## AIO-3399Pro-JD4 产品编译方法

### HDMI 显示编译

```
./FFTools/make.sh  -d rk3399pro-firefly-aiojd4 -j8 -l rk3399pro_firefly_aiojd4-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399pro_firefly_aiojd4-userdebug
```
## 其他安卓版本
* <font color=#ff0000 size=3>主要维护：</font>

   [《编译 Android9.0 固件》](compile_android9.0_firmware.md)  
* 支持但不维护：

   [《编译 Android8.1 固件》](compile_android8.1_firmware.md)  