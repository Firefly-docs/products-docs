### 下载 Android SDK

由于 Android SDK 源码包比较大,可以通过如下方式获取Android9.0源码包：
[下载链接](http://www.t-firefly.com/doc/download/page/id/54.html#other_333)

下载完成后，在解压前先校验下 MD5 码：
```
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.001
$ md5sum /path/to/rk3399pro_firefly_android9.0_20191126.7z.002

6e1d3969c8a0f643522727ff07800bb5  rk3399pro_firefly_android9.0_20191126.7z.001
a6b8d6a775c3d5ed28f4d41cb210a84d  rk3399pro_firefly_android9.0_20191126.7z.002
```

然后解压：
```
cd ~/proj/
7z x ./rk3399pro_firefly_android9.0_20191126.7z.001 -oAIO-3399Pro
cd ./AIO-3399Pro
git reset --hard
```



以下为从 gitlab 处更新的方法：：
```
#1. 进入SDK根目录
cd ~/proj/AIO-3399Pro

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399pro-pie-bundle.git .bundle

#3. 若下载仓库失败，目前bundle仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
#   可以从下方百度云链接下载并解压到SDK根目录，解压指令如下：

7z x rk3399pro-pie-bundle.7z  -r -o. && mv rk3399pro-pie-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```

也可以到如下地址在线查看源码：
[[https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#]](https://gitlab.com/TeeFirefly/firenow-oreo-rk3399#)
