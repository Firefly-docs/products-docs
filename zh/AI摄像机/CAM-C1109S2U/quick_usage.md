# 快速使用

本章讲述 ` CAM-C1126S2U/CAM-C1109S2U ` 配套使用虹软 ArcSoft 人脸识别算法 demo 示例软件的使用方法，不包含具体应用开发。

## 供电说明

由于 CAM-C1109S2U 设备内置了高功率 IR 红外补光灯。CAM-C1109S2U 必须接入主机主板出来的**机箱后置**的 `USB 3.0` 的接口。如果使用主机机箱前置的 `USB 2.0` 或者 `USB 3.0` 接口，可能会出现设备供电不足的情况。另外笔记本的 USB 接口也可能会出现供电不足的情况。**在这种供电电流不足情况下，设备可能会一直重启或者断开与上位机的连接。**

不同应用场景的供电解决办法：

1. 如果是 Facial_Gate 的应用场景，不需要通过 usb 与上位机进行数据通信的情况下。可以使用较大功率的电源给设备供电。如使用 5V 2A 或者 5V 3A 的电源适配器给设备供电。
2. 如果是需要通过 usb 与上位机进行数据通信的情况，则使用可以外接电源的 usb hub 来给 CAM-C1109S2U 设备电源供电。

## 应用场景介绍

虹软 ArcSoft 人脸识别算法套件 demo 示例软件分为两类：

1. Facial_Gate （人脸识别闸机）应用场景。该应用场景的工作原理是 `CAM-C1126CS2U/CAM-C1109S2U` 从摄像头获取视频帧数据，在本地进行 AI 人脸识别算法推理，将推理结果与本地数据库预留的人脸图片进行比对。然后将识别结果呈现到 MIPI 屏幕上。
2. AI_UVC （AI UVC摄像头）应用场景。该应用场景的工作原理是从将 `CAM-C1126CS2U/CAM-C1109S2U` 当作 UVC 摄像头，并通过 USB RNDIS 协议将 USB 接口模拟成一个网卡设备，这样上位机与 `CAM-C1126CS2U/CAM-C1109S2U` 就可以使用网络协议进行通信，如 socket 通信。 `CAM-C1126CS2U/CAM-C1109S2U` 从摄像头获取到视频帧数据，在本地进行 AI 人脸识别算法推理，将推理结果通过 usb 网卡传输到上位机。由上位机对推理结果处理，并且通过屏幕显示的方式呈现识别结果。该应用场景无需接入 MIPI 屏幕。

<font color=red>**重要说明：**</font>

1. 虹软 ArcSoft 提供的 Facial_Gate 和 AI UVC 的软件均为演示示例 demo ，没有源代码，也没有虹软对应的技术支持。它是用来展示虹软人脸识别算法的。并不能用作商用。由于是演示实例 demo 软件，所以可能存在一些 bug 。如果在使用 demo 的过程中遇到一些 bug ，如 ArcAICamera 上位机可能会出现：**插入人脸数据库失败**的问题。请尝试重新烧写固件再重新操作验证，中间最好不要出现操作错误，以防触发演示示例软件 bug 。

2. 虹软 ArcSoft 提供人脸识别算法的底层 API 技术支持。

3. CAM-C1109S2U 设备出厂固件为 `AI_UVC` 固件。

4. 如果使用时遇到问题请先尝试更新固件。如：修复了部分 Windows 系统` RNDIS `网卡无法识别或者出现`黄色感叹号`问题，如果设备出现此现象可以先尝试更新固件。

## Facial_Gate 固件

首次使用需要接入上位机进行部署调试，请参考方式一。上位机完成部署后，可使用方式二进行设备使用。 Facial_Gate 应用场景需要接入 MIPI 屏幕使用。

方式一：上位机部署调试步骤：

1. 设备接入 MIPI 屏幕。MIPI 屏幕的连接方法请参考：[《Display使用》](usage_display.md)。

2. 设备接上上位机主板出来的` USB 3.0 `接口，使用 TYPE-C 线供电开机。

3. 上位机搜索到 ADB 设备则代表设备已经正常开机。

4. 开机后 Windows 主机会识别到一张 USB 网卡，设置该网卡静态 IP 地址属性如下：

   * IP 地址：` 172.16.110.5 ` 
   * 子网掩码： `255.255.255.0` 
   * 网关： `172.16.110.1` 
   * 首选 DNS ：`8.8.8.8`

5. 打开浏览器地址栏输入` 172.16.110.2 `可进入 Web 后端管理平台。PS：如果驱动网卡设备不正常可以使用驱动精灵等第三方驱动安装工具安装或者更新驱动。

6. 首次使用 Web 后端管理平台需要设置登录密码：

   ![](../../../rv1126_img/CAM-C1109S2U/ArcFaceGo_login.png)

7. 登录后，选择设备激活。激活码请联系业务购买。

   ![](../../../rv1126_img/CAM-C1109S2U/ArcFaceGo_activate.png)

8. 激活设备后在 Web 后端管理平台上点击：[人员管理-->人员注册] 来进行人脸注册操作。进行人脸注册操作。

9. 完成人脸注册后，人脸面对摄像头，可以看到 MIPI 屏幕输出人脸识别结果。

方式二：不接入上位机使用步骤：

1. 设备接入 MIPI 屏幕。MIPI 屏幕的连接方法请参考：[《Display使用》](usage_display.md)。
2. 将 TYPE-C 单独接到 5V/2A 的电源。
3. 人脸面对摄像头，可以看到 MIPI 屏幕输出人脸识别结果。



说明：

设备外接了 MIPI 屏幕会先看到 Firefly 的开机 logo，稍等片刻界面进入预置的闸机应用，如果闸机应用未授权，会显示未授权或授权失败无法进行人脸识别。

![](../../../rv1126_img/CAM-C1109S2U/authorization.png)



## AI_UVC 固件

AI_UVC 根据使用平台不同，需要安装不同的上位机 APP 工具。虹软提供了 Windows 上位机 APP 工具和 Android 上位机 APP 工具，没有提供 Linux 上位机 APP  工具。

` ArcAICameraSetup.exe `、` Android APK `和使用说明都可以在此仓库下载获取：

[https://gitlab.com/firefly-linux/external/arcuvc_app.git](https://gitlab.com/firefly-linux/external/arcuvc_app.git)

### Windows

官方测试使用 Windows10 系统。测试步骤如下：

1. 点击下载 Windows 上位机工具：[ArcAICameraSetup](https://www.t-firefly.com/doc/download/99.html#other_798) 。

2. CAM-C1109S2U 接入 Windows 主机主板出来的机箱后置` USB 3.0 `接口。

3. 正常开机后红外补光灯会亮起。

4. 正常开机后 Windows 主机会识别到一张 USB RNDIS 网卡。Windows 主机设置该网卡静态 IP 地址属性如下：
   * IP 地址：` 172.16.110.5 ` 
   * 子网掩码： `255.255.255.0` 
   * 网关： `172.16.110.1` 
   * 首选 DNS ：`8.8.8.8`

5. 设置完成可以和 CAM-C1109S2U 设备进行网络通讯。PS：如果驱动网卡设备不正常可以使用驱动精灵等第 3 方驱动安装工具安装或者更新驱动。

6. 安装 Windows 上位机工具` ArcAICameraSetup.exe `并打开，按提示进行授权后可以使用内置 AI_UVC 应用。PS：请联系业务购买授权码。

7. 设置界面，点击右上角按钮可以进入人脸注册：</br>

   ![](../../../rv1126_img/CAM-C1109S2U/arcsetting.png)

8. 识别成功，注册成功后可以回到预览页面查看效果：</br>
   ![](../../../rv1126_img/CAM-C1109S2U/arcuvc_face.png)

9. 识别记录，识别到人脸后会记录在后台：</br>
   ![](../../../rv1126_img/CAM-C1109S2U/arcuvc_log.png)

### Android

官方测试使用 RK3399 Andorid 7.1。测试步骤如下：

1. 点击下载 Android 上位机 APP 工具：[Android APK](https://www.t-firefly.com/doc/download/99.html#other_799) 。

2. 接上 Android 设备的 ` USB 3.0 `接口。

3. 正常开机后红外补光灯会亮起。

4. 下载安装` ArcSoft_AICamera_xxx.apk `并且打开应用,按提示进行授权后可以使用内置 AI_UVC 应用。PS：请联系业务购买授权码。

5. 授权，获取设备信息并且联系我们根据设备信息生成离线授权文件：</br>

   ![](../../../rv1126_img/CAM-C1109S2U/androi_activation.png)

6. 预览，授权成功后回到预览画面可以看到人脸识别功能已经被激活：</br>

   ![](../../../rv1126_img/CAM-C1109S2U/android_detect1.png)

7. 设置，点击预览界面右上方齿轮可以进入设置界面，默认管理员密码为 `123456` ：</br>
   ![](../../../rv1126_img/CAM-C1109S2U/android_password.png)

8. 更换自己公司的` logo `和`公司名`：</br>

   ![](../../../rv1126_img/CAM-C1109S2U/androi_info.png)

9. 根据自己的需求配置自己的识别参数：</br>
   ![](../../../rv1126_img/CAM-C1109S2U/android_setting.png)

10. 点击右上角人脸注册注册自己第一张人脸：</br>
    ![](../../../rv1126_img/CAM-C1109S2U/androi_register.png)

11. 注册成功后，回到预览界面查看效果：</br>
    ![](../../../rv1126_img/CAM-C1109S2U/android_detect2.png)

12. 可以进入设置浏览识别成功和失败记录：</br>
    ![](../../../rv1126_img/CAM-C1109S2U/androi_log1.png)

    ![](../../../rv1126_img/CAM-C1109S2U/androi_log2.png)

## 应用授权

无论使用` Windows `还是` Android `进行激活，都会进入到类似以下界面。请点击`采集设备信息`把提示生成的文件拷贝出来并且保存。并且把该文件发送给业务然后联系业务购买授权。业务会提供对应的设备`离线激活文件`，您可以使用该文件激活设备。

<font color=red>**重要说明：**</font>

1. 对于需要使用虹软人脸识别算法底层 API 的客户。请跟业务索要激活授权的：APP_ID 、SDK_KEY 和 ACTIVE_KEY 信息。

2. 激活文件和设备是`一一对应`的，获取到激活文件后请务必保存好。

3. 正常升级官方固件后可以使用对应激活文件`重新激活`。

4. **激活设备后绝对不可以进行`擦除`或者其他修改 `backup` 分区的操作，否则需要重新购买授权激活。**

![](../../../rv1126_img/CAM-C1109S2U/androi_activation.png)

## 设备调试

`CAM-C1126S2U/CAM-C1109S2U ` 设备可以使用` TYPE-C OTG ` 线缆和`串口`进行设备调试。

### USB 线缆调试

#### 硬件接线

1. 准备一条 TYPE-C 线。TYPE-C 线一端接电脑的 USB 口，另一端接 USB OTG 调试接口。接线如下图：
![](../../../rv1126_img/CAM-C1109S2U/upgrade_otg_interface.jpg)
2. 基于你的系统安装 ADB。

#### Windows 下的 ADB 安装：

Windows版本下载链接：https://dl.google.com/android/repository/platform-tools-latest-windows.zip

1. 解压 `platform-tools-latest-windows.zip` 到自定义目录
2. 按键 `windows + r` 打开运行，输入 `sysdm.cpl` ，回车。高级-->环境变量-->系统变量-->双击 Path -->新建
3. 例：设置如下图所示：
   ![](../../../rv1126_img/CAM-C1109S2U/set_path.png)
4. 注：路径需要改成你解压的自定义目录
5. 以管理员权限打开命令提示符，此时电脑 USB 接口连接了 RV1126 的 USB 调试口。命令提示符终端输入 `adb devices`
   ```bash
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
   ```shell
   sudo apt-get update
   sudo apt-get install android-tools-adb
   ```

2. 验证安装：
   ```shell
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

CAM-C1109S2U 设备需要设置静态 IP 地址来进行网口调试。该调试方法需要建立在用户手动设置静态 IP 之后才能进行。

注意：AI_UVC 固件默认没有打开用于 SSH 登录终端的服务。只能使用 ADB 和串口调试。如果需要 SSH 登录终端的服务，需要在 buildroot 配置里面进行打开。

#### 网络 ADB 调试
1. 设备接入 TYPE-C 接口
2. 连接设备网络 ADB。假设 CAM-C1109S2U 设备设置的静态 IP 地址为：172.16.110.5 。则执行以下命令进行连接：
   ```bash
   adb connect 172.16.110.5
   ```

3. 使用 ADB 进行调试。
   ```
   adb shell
   ```

#### ssh 调试

1. ssh 登录到开发板进行调试。

2. 2021 年 9 月份之前版本的固件默认账户为：root，默认密码为：rockchip。

3. 2021 年 9 月份之后版本的固件默认账户为：root，默认密码为：firefly。

4. 假设 CAM-C1109S2U 设备设置的静态 IP 地址为：172.16.110.5 。则执行以下命令进行连接：
   ```bash
   ssh root@172.16.110.5
   ```

   Debian10 系统连接方式如下。默认账户为：firefly，默认密码为：firefly。
   ```
   ssh firefly@172.16.110.5
   ```

### 串口调试

1. 准备 USB 转串口模块，推荐使用 Firefly 的串口模块。

2. USB 串口模块连接主机 USB 口和设备串口。

3. 设置波特率为 `1.5 M` 。

4. 根据你的系统下载、安装、打开系统平台支持的串口调试工具（Ubuntu 推荐使用 minicom，Window 推荐使用 putty ）。如果有串口调试工具有软硬件流控的配置项请关闭该功能。

5. 串口接线说明：如下图从上往下依次是 `RX, TX, GND`。

   ![](../../../rv1126_img/CAM-C1109S2U/cam_debug_instruction.png)