# 使用方法

本章讲述讲述` CORE-1126-JD4/CORE-1109-JD4 `配套官方底板的使用方法，不包含具体应用开发，应用开发请看[《软件开发》](Source_code.html)一节。

## 上电开机

设备出厂默认自带 AI-IPC 场景固件。

* 接入` DC 12V `电源，或者` POE `供电。

* 开机后以太网旁边两个 LED 灯会常亮。

* 设备接入网线，确保和调试使用的 `windows` 电脑处于同一局域网下。

* windows 下载 [RK_IPCamera_Tool-Vx.x.zip](http://www.t-firefly.com/doc/download/101.html)，解压运行软件。

* 使用软件搜索 AI 网络摄像头设备 IP。

![](../../../rv1126_img/iptool.png)

* 在浏览器上输入设备 IP 地址可以进入管理页面，默认帐号密码都为 `admin`。

![](../../../rv1126_img/login.png)

* 使用 RTSP 流播放器可以预览摄像头。

![](../../../rv1126_img/vlc.png)

* 如果外接了 MIPI 显示屏，显示屏上会出现摄像头预览画面。

## 快速启动
目前 CORE-1126-JD4 最新的 SDK 已适配快速启动。编译快速启动固件选项：
```
# 编译 CORE-1126-JD4 V1.1 快速启动固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45-tb-v11.mk
```
快速启动固件启动后，MIPI DM-M10R800 V2 屏幕模组可预览摄像头画面。若接入网络，还可预览摄像头推流 rtsp。Linux PC 预览命令：
```
vlc rtsp://<设备 ip 地址>
```
注意：该快速启动固件`仅适用于 CORE-1126-JD4 V1.1 版本的核心板`。核心板版本详见硬件上的丝印。

## 设备调试

 设备可以使用 `USB 线缆` 、`网口` 或者 `串口` 进行调试。

### USB 线缆调试

#### 硬件接线

* 准备一条公对公的 USB 线

![](../../../rv1126_img/usb.jpeg)

* USB 线一端接电脑的 USB 口，另一端接 RV1126/RV1109 底板的 USB OTG 调试接口。接线如下图：

![](../../../rv1126_img/usb_connect.jpeg)

#### USB ADB 调试

* 使用网口线连接设备，使设备和调试主机处于同一局域网 。
* 基于你的系统安装 ADB。

#### Windows 下的 ADB 安装：
Windows版本：https://dl.google.com/android/repository/platform-tools-latest-windows.zip
* 解压到自定义目录
* 按键 `windows + r` 打开运行，输入 `sysdm.cpl` ，回车。高级-->环境变量-->系统变量-->双击 Path -->新建
* 例：设置如下图所示：

![](../../../rv1126_img/set_path.png)

* 注：路径需要改成你解压的自定义目录

* 以管理员权限打开命令提示符，此时电脑 USB 接口连接了 RV1126 的 USB 调试口。命令提示符终端输入 `adb devices` 
```
C:\Users\lvsx>adb devices
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached
d70a14329414b400        device
```
* 使用 ADB 进行调试。
```
adb shell
```

#### Ubuntu 下的 ADB 安装
```
sudo apt-get update
sudo apt-get install android-tools-adb
```

* 验证安装：
```
lvsx@lvsx:~$ adb devices
* daemon not running; starting now at tcp:5037
* daemon started successfully
List of devices attached
d70a14329414b400	device
```
* 使用 ADB 进行调试。
```
adb shell
```

### 网口调试

* 通过` RK_IPCamera_Tool `查看设备 IP。 RK_IPCamera_Tool 工具可到资源下载页面下载。

#### 网络 ADB 调试
* 使用网线接入任意网络接口
* 网线连接设备网络 ADB。
```
adb connect 192.168.22.33
```
* 使用 ADB 进行调试。
```
adb shell
```
#### ssh 调试
* ssh 登录到开发板进行调试
* 2021 年 9 月份之前版本的固件默认账户为：root，默认密码为：rockchip。
* 2021 年 9 月份之后版本的固件默认账户为：root，默认密码为：firefly。
```
ssh root@192.168.22.33
```

* Debian10 系统默认账户为：firefly，默认密码为：firefly。
```
ssh firefly@192.168.22.33
```

### 串口调试

* 准备 USB 转串口模块，推荐使用 Firefly 的串口模块。
* 安装串口模块。
* USB 串口模块连接主机 USB 口和设备串口。
* 设置波特率为` 1.5 M `。
* 根据你的系统下载、安装、打开系统平台支持的串口调试工具（Ubuntu 推荐使用 minicom，Window 推荐使
用 putty ）。如果有串口调试工具有软硬件流控的配置项请关闭该功能。
* Firefly 的串口模块硬件连接如图所示：

![](../../../rv1126_img/DEBUG.jpg)


## 摄像头
RV1126/RV1109 核心板 + 底板已适配配套的 400W HDR 高动态红外增强广角 [OS04A10](https://item.taobao.com/item.htm?spm=a230r.1.14.1.422453ectFC4JL&id=648680997969&ns=1&abbucket=18#detail) MIPI 接口摄像头。

![](../../../rv1126_img/os04a10.jpg)

底板与摄像头接法如图所示：

![](../../../rv1126_img/camera.jpg)

* 注：摄像头上的红线和黑线用作切换滤光片使用。
* 命令切换滤光片
```
# 查看 SDK 版本
cd path-to-sdk/
realpath .repo/manifest.xml
```
* 若上述命令获取到的 SDK 版本`是` rv1126_rv1109_linux_release_20220324_v2.2.5b.xml 版本及日期为 2022.03.24 以上版本的则使用以下命令进行 IRCUT （滤光片切换）：
```
# 关闭滤光片
v4l2-ctl -d /dev/v4l-subdev5 --set-ctrl 'band_stop_filter=0'

# 打开滤光片
v4l2-ctl -d /dev/v4l-subdev5 --set-ctrl 'band_stop_filter=1'
```

* 若上述命令获取到的 SDK 版本`不是` rv1126_rv1109_linux_release_20220324_v2.2.5b.xml 版本及日期为 2022.03.24 以上版本的则使用以下命令进行 IRCUT （滤光片切换）：
```
# 关闭滤光片
v4l2-ctl -d /dev/v4l-subdev4 --set-ctrl 'band_stop_filter=0'

# 打开滤光片
v4l2-ctl -d /dev/v4l-subdev4 --set-ctrl 'band_stop_filter=1'
```

## MIPI 屏幕
* RV1126/RV1109 配套的屏幕为[10.1 寸 IPS 全视觉屏幕模组 MIPI 多点触摸 1280x800 像素](https://item.taobao.com/item.htm?spm=a1z10.5-c-s.w4002-24147662620.19.187cce42Nq5fe8&id=655100190974)。MIPI 屏幕如图所示：

![](../../../rv1126_img/screen.jpg)

* 底板与 MIPI 屏幕的硬件连接方式如图所示：

![](../../../rv1126_img/mipi_screen.jpg)

* 注：默认公版固件在底板不接摄像头的情况下只显示 firefly LOGO 图标。如果底板接了 0S04A10 摄像头，则屏幕显示摄像头预览界面。