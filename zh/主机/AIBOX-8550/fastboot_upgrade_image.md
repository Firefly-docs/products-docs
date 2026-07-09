# 升级分区镜像

## 进入 Bootloader 升级模式

在设备正常运行的情况下，使用 Type-A 转 C USB 数据线 连接好设备的烧录口和电脑，然后在设备终端或调试串口上运行：

```shell
sudo systemctl reboot bootloader
```

![](../../../qcom_img/AIBOX-8550/download_port.jpg)

### 确认是否成功

打开 windows 终端 (PowerShell)，前往之前下载的 Android Platform Tools 的位置，执行 ".\fastboot.exe devices"

如果设备成功进入 bootloader 模式，此时 fastboot 工具应该会显示一个设备：

```
PS D:\path\to\platform-tools> .\fastboot.exe devices
xxxxxxxx         fastboot
```

如果显示设备，则检查驱动是否安装成功、USB 线连接是否正确。

## 烧写分区

烧录方法非常简单，只需要 fastboot.exe flash <分区名> <镜像文件位置>，比如

```
.\fastboot.exe flash boot D:\path\to\boot.img

.\fastboot.exe flash dtbo D:\path\to\dtbo.img

# 烧录完成后执行下面命令重启设备
.\fastboot.exe reboot
```

注意：烧录错误的文件会导致系统故障，所以烧录前请确认分区和文件正确。如果不明白可以先寻求帮助，不要盲目操作。