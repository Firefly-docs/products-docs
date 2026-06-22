

### 下载 SDK
* SDK通过邮件的方式获取，把订单号发送到<font color=red>sales@t-firefly.com</font>邮箱并注明需要的SDK名称  
[firefly_rk3588_android12_git_20240704]()

* 下载完成后，在解压前先校验下 MD5 码：

    ```
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.001
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.002
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.003
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.004
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.005
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.006
    $ md5sum /path/to/firefly_rk3588_android12_git_20240704.7z.007

    1ce2a5ab4e3a9f0a4d6d8cd47ca4981d  firefly_rk3588_android12_git_20240704.7z.001
    553fb5ee2cc5623d97bfb19ba299605b  firefly_rk3588_android12_git_20240704.7z.002
    cb9cce227a79356fa57c0781a426fd89  firefly_rk3588_android12_git_20240704.7z.003
    1461a09bacbef0cb3c9a9008e8483e93  firefly_rk3588_android12_git_20240704.7z.004
    d154b39087bbf410a7973da97a6a9ecb  firefly_rk3588_android12_git_20240704.7z.005
    7cfc42443caa57cb4cf474bf3e78b5ce  firefly_rk3588_android12_git_20240704.7z.006
    9ad4374d89212a882dc00541769732f8  firefly_rk3588_android12_git_20240704.7z.007
    
    ```

* 然后解压：

    ```
    cd ~/proj/
    7z x ./firefly_rk3588_android12_git_20240704.7z.001 -oRK3588_Android12.0
    cd ./RK3588_Android12.0
    git reset --hard
    ```
<font color=red>**注意：不要在共享文件夹、挂载文件夹以及非英文目录解压SDK，避免产生不必要的错误**</font>

### 更新 SDK
下载 SDK 后，从 gitlab 处更新代码的方法：
```
#1. 进入 SDK 根目录
cd ~/proj/RK3588_Android12.0

#2. 下载远程 bundle 仓库
git clone https://gitlab.com/T-Firefly/rk3588-android12.0-bundle.git .bundle

#3. bundle仓库会随着更新的资源越多而会越来越大，如果bundle仓库下载速度缓慢或若下载失败，
#   请在资源下载界面选择对应的机器bundle文件进行下载并解压到SDK根目录，解压指令如下:

7z x rk3588-android12.0-bundle.7z  -r -o. && mv rk3588-android12.0-bundle .bundle

#4. 更新 SDK，并且后续更新不需要再次拉取远程仓库，直接执行以下命令即可
.bundle/update

#5. 按照提示已经更新内容到 FETCH_HEAD,同步 FETCH_HEAD 到 firefly 分支
git rebase FETCH_HEAD
```

下载页面选择云盘下载 [Android12.0 Bundle]()。