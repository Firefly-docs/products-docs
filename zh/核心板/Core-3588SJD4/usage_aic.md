## 容器虚拟安卓

AIC (Android in Container) 是指在 Linux 系统中使用容器来运行安卓。RK3588 Linux 即可使用 docker 来运行安卓并支持多开。

集群服务器搭配 AIC 可以增加安卓的密度，在云手机、云游戏等场景有很大的帮助。

此处有固件可供测试：[点击前往](https://pan.baidu.com/s/1sbZwOc4peZfn0HDX9O0lsg) 提取码 1234

### 使用方法

说明：

1. 固件使用的宿主系统是 Ubuntu Minimal 版本，使用 debug 串口、ssh、adb shell 等方式进行交互，没有桌面显示。
2. 安卓的交互是通过网络 adb 连接并配合 scrcpy 等投屏工具实现，不是接入鼠标键盘来操作。
3. 操作 AIC 需要知道基础的 Linux 终端操作知识、docker 容器知识、adb 使用方法。

#### 创建容器

将下载到的 container 文件夹复制到宿主机(RK3588)的 /userdata/ 目录下，通过里面的 aic.sh 脚本进行操作。

请将设备联网，首次运行需要先初始化：
```bash
cd /userdata/container
./aic.sh -i
```

等待初始化完成后，将安卓镜像也复制到 /userdata/container/ (目前已经内置了一个安卓镜像)

部署容器，执行./aic.sh -r <安卓镜像.tgz> <容器数量>
```bash
./aic.sh -r rk3588_docker-android12-userdebug-super.img-20231128.1932.tgz 3
```

网络默认是采用端口映射方式。
```
<宿主ip>:1100 --> <容器0>:5555
<宿主ip>:1101 --> <容器1>:5555
......
```

更多内容请阅读 aic.sh 脚本

#### 连接容器

在局域网中使用任意电脑通过 adb 即可访问容器。
```bash
adb connect <host ip>:<port>

# 在上一步连接成功后
# 投屏显示
scrcpy -s <host ip>:<port>
```

adb 和 scrcpy 的安装方法请自行上网搜索。

#### 管理容器

使用 docker 命令管理容器即可。
```bash
# 查看所有容器
root@firefly:~# docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED      STATUS                    PORTS     NAMES
cad5a331dea9   rk3588:firefly   "/init androidboot.h…"   6 days ago   Exited (137) 6 days ago             android_1
37f60c3b6b80   rk3588:firefly   "/init androidboot.h…"   6 days ago   Up 13 seconds                       android_0

# 启动/停止容器
docker start/stop <NAMES>

# 连接到安卓终端(再次执行 exit 退出)
docker exec -it <NAMES> sh

# 删除容器
docker rm <NAMES>
```

### 性能展示

以 root 身份执行以下命令开启性能模式可以获得更好的体验：
```bash
# 出现一个 Invalid argument 是正常的，不用管
root@firefly:~# echo performance | tee $(find /sys/devices -name *governor)
performance
tee: /sys/devices/system/cpu/cpuidle/current_governor: Invalid argument
```

进入安卓的终端或使用 adb shell 执行如下命令可以打印游戏渲染帧率：
```bash
# 注意，只有在游戏运行中，这个命令才会打印帧率
setprop debug.sf.fps 1;logcat -s SurfaceFlinger
```

单个 RK3588 使用 AIC 同时运行两个最高画质原神可达 35 帧以上:
![](../../../rk3588_img/common/aic_ys_performance.png)