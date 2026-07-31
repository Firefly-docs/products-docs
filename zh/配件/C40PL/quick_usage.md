# 快速使用

本章讲述讲述` CORE-1126-JD4/CORE-1109-JD4 `配套官方底板的使用方法，不包含具体应用开发。

## 上电开机

设备出厂默认自带 AI-IPC 场景固件。
1. 接入` DC 12V `电源，或者` POE `供电。
2. 开机后以太网旁边两个 LED 灯会常亮。
3. 设备接入网线，确保和调试使用的 `windows` 电脑处于同一局域网下。要求电脑和 1126 板子是同一个网段。
4. windows 下载 [RK_IPCamera_Tool-Vx.x.zip](http://www.t-firefly.com/doc/download/101.html)，解压运行软件。
5. 使用软件搜索 AI 网络摄像头设备 IP。
![](../../../rv1126_img/C40PL/iptool.png)
6. 在浏览器上输入设备 IP 地址可以进入管理页面，默认帐号密码都为 `admin`。
![](../../../rv1126_img/C40PL/login.png)
7. 使用 RTSP 流播放器可以预览摄像头。
![](../../../rv1126_img/C40PL/vlc.png)
8. 如果外接了 MIPI 显示屏，显示屏上会出现摄像头预览画面。

## 设备调试

设备可以使用 `USB 线缆` 、`网口` 或者 `串口` 进行调试。

### USB 线缆调试

#### 硬件接线

1. 准备一条公对公的 USB 线
![](../../../rv1126_img/C40PL/usb.jpeg)
2. USB 线一端接电脑的 USB 口，另一端接 RV1126/RV1109 底板的 USB OTG 调试接口。接线如下图：
![](../../../rv1126_img/C40PL/usb_connect.jpeg)
3. 基于你的系统安装 ADB。

#### Windows 下的 ADB 安装：

Windows版本下载链接：https://dl.google.com/android/repository/platform-tools-latest-windows.zip

1. 解压 `platform-tools-latest-windows.zip` 到自定义目录
2. 按键 `windows + r` 打开运行，输入 `sysdm.cpl` ，回车。高级-->环境变量-->系统变量-->双击 Path -->新建
3. 例：设置如下图所示：
	![](../../../rv1126_img/C40PL/set_path.png)
4. 注：路径需要改成你解压的自定义目录

5. 以管理员权限打开命令提示符，此时电脑 USB 接口连接了 RV1126 的 USB 调试口。命令提示符终端输入 `adb devices` 
```
C:\Users\lvsx>adb devices
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached
d70a14329414b400        device
```
6. 使用 ADB 进行调试。
```
adb shell
```

#### Ubuntu 下的 ADB 安装
1. apt-get 安装 android-tools-adb 包
```
sudo apt-get update
sudo apt-get install android-tools-adb
```

2. 验证安装：
```
lvsx@lvsx:~$ adb devices
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached
d70a14329414b400	device
```
3. 使用 ADB 进入设备调试。
```
adb shell
```
4. 使用 ADB 拷贝 linux pc 文件到 rv1126 设备的 userdata 目录。
```
adb push /path/to/your-file /userdata/
```
5. 使用 ADB 拷贝 rv1126 设备文件到 pc 本地目录。rv1126-file 更换为实际的文件名称。
```
adb pull /path/to/rv1126-file ./
```

### 网口调试

通过` RK_IPCamera_Tool `查看设备 IP。 RK_IPCamera_Tool 工具可到资源下载页面下载。

#### 网络 ADB 调试
1. 使用网线接入任意网络接口
2. 网线连接设备网络 ADB。
```
adb connect 192.168.22.33
```
3. 使用 ADB 进行调试。
```
adb shell
```

#### ssh 调试

1. ssh 登录到开发板进行调试
2. 2021 年 9 月份之前版本的固件默认账户为：root，默认密码为：rockchip。
3. 2021 年 9 月份之后版本的固件默认账户为：root，默认密码为：firefly。
```
ssh root@192.168.22.33
```

* Debian10 系统默认账户为：firefly，默认密码为：firefly。
```
ssh firefly@192.168.22.33
```

### 串口调试

1. 准备 USB 转串口模块，推荐使用 Firefly 的串口模块。
2. 安装串口模块。
3. USB 串口模块连接主机 USB 口和设备串口。
4. 设置波特率为` 1.5 M `。
5. 根据你的系统下载、安装、打开系统平台支持的串口调试工具（Ubuntu 推荐使用 minicom，Window 推荐使用 putty ）。如果有串口调试工具有软硬件流控的配置项请关闭该功能。
6. Firefly 的串口模块硬件连接如图所示：

![](../../../rv1126_img/C40PL/DEBUG.jpg)