## 下载 Android SDK

由于 SDK 较大，可以去下载页面选择云盘下载 **Firefly-RK3399_Android10.0_git_20211222.7z**：

[下载链接](http://www.t-firefly.com/doc/download/page/id/54.html#other_399)  

注：源码与bundle压缩包均存放在云盘中

下载完成后，在解压前先校验下 MD5 码：
```
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.001
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.002
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.003
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.004
$ md5sum /path/to/Firefly-RK3399_Android10.0_git_20211222.7z.005

fb41fcdc48b1cf90ecac4a5bb8fafc7a  Firefly-RK3399_Android10.0_git_20211222.7z.001
82d665fb54fb41245d9317312e7b7578  Firefly-RK3399_Android10.0_git_20211222.7z.002
0dd40827976200a6fb61623c9e87fe7c  Firefly-RK3399_Android10.0_git_20211222.7z.003
32324c3afe615394ee4cdf842c53ea8d  Firefly-RK3399_Android10.0_git_20211222.7z.004
9b1656ae07b4fe9ddf7d9ef2dfa2f95a  Firefly-RK3399_Android10.0_git_20211222.7z.005

```

然后解压：
```
cd ~/proj/
7z x ./Firefly-RK3399_Android10.0_git_20211222.7z.001 -ork3399_Android10.0
cd ./rk3399_Android10.0
git reset --hard
```



以下为从 gitlab 处更新的方法：：
```
#1. 进入SDK根目录
cd ~/proj/rk3399_Android10.0

#2. 下载远程bundle仓库
git clone https://gitlab.com/TeeFirefly/rk3399-android10.0-bundle.git .bundle

#3. 若下载仓库失败，目前bundle仓库占用空间较大，所以同步的时候可能会出现卡住或失败的问题，
#   可以从下方百度云链接下载并解压到SDK根目录，解压指令如下：

7z x rk3399-android10.0-bundle.7z  -r -o. && mv rk3399-android10.0-bundle/ .bundle/

#4. 更新SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步FETCH_HEAD到firefly分支
git rebase FETCH_HEAD
```