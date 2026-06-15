### 下载 Android SDK

由于 SDK 较大，可以取下载页面选择云盘下载 Firefly-RK3399_Android8.1_xxx.7za(可以直接使用Firefly-RK3399源码包)：
[Android8.1 源码包]

下载完成后，在解压前先校验下 MD5 码：
```
$ md5sum /path/to/Firefly-RK3399_Android8.1_git_xxx.7z
6d34e51fd7d26e9e141e91b0c564cd1f Firefly-RK3399_Android8.1_xxx.7z
```

然后解压：
```
mkdir -p ~/proj/roc-rk3399-pc
cd ~/proj/roc-rk3399-pc
7z x /path/to/Firefly-RK3399_Android8.1_git_xxx.7z -o./
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


[Android8.1 源码包]: https://www.t-firefly.com/doc/download/124.html#other_144