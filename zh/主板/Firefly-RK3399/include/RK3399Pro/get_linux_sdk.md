### 下载 Firefly_Linux_SDK 分卷压缩包

**由于 Firefly_Linux_SDK 源码包比较大，部分用户电脑不支持4G以上文件或单个文件网络传输较慢, 所以我们采用分卷压缩的方法来打包SDK。用户可以通过如下方式获取 Firefly_Linux_SDK源码包：**[Firefly_Linux_SDK源码包]

### 解压 Firefly_Linux_SDK 分卷压缩包

第一次使用SDK需执行3个步骤，如果是后续想更新SDK，只需执行第3步进行网络更新即可
```
1. 解压SDK

chmod +x ./sdk_tools.sh

创建一个目录以存放SDK：比如我现在这个是3588的SDK，我想解压到上一层文件夹，避免污染当前目录

mkdir ../firefly_rk3588_SDK
./sdk_tools.sh --unpack -C ../firefly_rk3588_SDK


2. 还原工作目录

选择刚才解压后的目录

./sdk_tools.sh --sync -C ../firefly_rk3588_SDK

可以使用上面脚本执行或者手动执行命令，选择其中一种即可

# 进入刚刚解压后的目录，比如我这里是../firefly_rk3588_SDK
cd ../firefly_rk3588_SDK
.repo/repo/repo sync -l
.repo/repo/repo start firefly --all


3. 更新SDK

前面2个步骤只在第一次解压SDK时执行，后续更新SDK只需进入SDK目录执行第3步骤，进行网络更新

.repo/repo/repo sync -c --no-tags
```

### 更新 Firefly_Linux_SDK

后续可以使用以下命令更新 SDK
```
.repo/repo/repo sync -c --no-tags
```

[下载链接]: http://www.t-firefly.com/doc/download/page/id/3.html
[Firefly_Linux_SDK源码包]: http://www.t-firefly.com/doc/download/page/id/3.html#other_423





