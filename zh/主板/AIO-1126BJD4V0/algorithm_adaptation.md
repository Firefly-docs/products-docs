# 算法适配

这一章主要针对两种用户：

（一）用户使用现成第三方模型算法。这时候用户不需要理会模型训练、转换和推理的细节，只需要完成算法的部署就可以了。

（二）用户自己实现了算法、训练了模型。用户需要查看[《NPU 使用》](usage_npu.md)一节，把自己的模型转换为` CAM-C1126S2U/CAM-C1109S2U `设备中的 NPU 可用的` RKNN `模型来使用。本章以虹软和 Rockchip 算法部署开发为例进行介绍。

## 适配虹软算法

虹软提供了两种开发包

1. 提供最基础的 SDK 。这里把它叫做`纯净版` 。

2. 提供完整的人脸识别管理系统。这里把它叫做`完整版`。

纯净版和完整版的区别：

纯净版只包含了人脸识别所需库，用户可以根据自己的需求设计 UI 和业务逻辑。

完整版包含了人脸识别闸机 UI 和人脸后端管理应用，大大的方便了项目开发。

### 纯净版

1. 点击下载 ArcFace Linux RV1109 SDK 版本软件包：[ArcFace_RV1109_SDK.zip](https://www.t-firefly.com/doc/download/99.html#other_416)

2. 解压 ArcFace_RV1109_SDK.zip

3. 参考[《快速使用》](quick_usage.md)完成设备联网。如果使用离线授权可以跳过此步。

4. 参考软件包 API 文档完成设备激活和应用开发。使用纯净版开发技术要求比较高，需要熟悉虹软人脸识别算法 API 的调用。

### 完整版

官方固件默认内置完整版，所以直接开机使用就可以了，以下步骤除了可以用于完整版部署外，还可以用于更新版本。

1. 点击下载 ArcFace Linux RV1109 版本软件包，[ArcFaceGo_RV1109_App.zip](https://www.t-firefly.com/doc/download/99.html#other_416)。

2. 解压 ArcFaceGo_RV1109_App.zip。

3. 确保设备和主机连接且 ADB 可用。

4. 执行脚本完成安装和部署。

   ```shell
   chmod 777 firefly.sh
   ./firefly.sh
   ```
5. ADB 执行闸机应用启动脚本，此时接屏可以看到程序运行，但是设备未激活无法进行人脸识别。
   ```
   adb shell ./userdata/arc/start_app.sh 90
   ```
6. 上位机设置 USB 网卡静态 IP 为 172.16.110.5。
7. 浏览器中打开 172.16.110.2 进入 Web 端。
8. 参考[《快速使用》](quick_usage.md)完成设备联网。如果使用离线授权可以跳过此步。
9. 登录 Web 参考软件包中文档完成激活。具体应用使用方法也可以参考文档。


## 适配 Rockchip 算法

Rockchip 自主研发了一套人脸识别算法名为` rockface `。在` rockface `的基础上封装了摄像头预览功能、语音播报、人脸识别 UI 部分显示功能和人脸注册管理功能等。这套基于 rockface 开发的应用叫做` rkfacial `。这里同样也把` rockface ` 称为`纯净版`，`rkfacial `称为`完整版`。

### 代码位置

1. rockface
	* 路径：/path/to/sdk/external/rockface
	* 说明：rockface 人脸 SDK 提供一系列人脸识别分析相关功能，充分利用了 NPU 对算法模型进行加速。开发者通过 SDK 提供的 API 接口能够快速构建人脸 AI 应用。
2. rkfacial
	* 路径：/path/to/sdk/external/rkfacial  
	* 说明：rockface API 参考指南。
3. QFacialGate
	* 路径：/path/to/sdk/app/QFacialGate
	* 说明：最终运行的 QT 应用程序

### 配置编译

Rockchip 已经把代码添加到` buildroot `源码编译中，只需要在` buildroot `编译时打开或者关闭对应配置即可。

```
BR2_PACKAGE_QFACIALGATE=y
BR2_PACKAGE_RKFACIAL_USE_WEB_SERVER=y 
BR2_PACKAGE_ROCKFACE_FACE_DETECTION_V3_FAST=y   
BR2_PACKAGE_ROCKFACE_LIVING_DETECTION=y
```

除了以上配置外还有一些依赖配置，详细请参考` configs/firefly_rv1126_rv1109_facial_gate_defconfig `配置文件。

### QT 开发

`QFacialGate `是程序的入口。由于出厂固件内置 Rkfacial 试用版，你可以直接修改、编译、替换 QFacialGate 就可以完成前期开发。

* 在完成一次 buildroot 完整编译后进入编译输出目录对 QFacialGate 二次开发。
```
# 这里是使用默认配置编译后输出的路径
cd sdk/buildroot/output/firefly_rv1126_rv1109_facial_gate/build/QFacialGate-1.0/ 

# 执行编译
make
```

* 可以在识别成功后添加自己的一些控制，如外接的继电器、LED灯。
```
# desktopview.cpp -> paintBox()
if(ret) {
        display_paint_box(left, top, right, bottom);
        switch(desktopView->videoItem->facial.state) {
                case USER_STATE_REAL_UNREGISTERED:
                        display_set_color(set_yuv_color(COLOR_B));
                        break;
                case USER_STATE_REAL_REGISTERED_WHITE:
                        display_set_color(set_yuv_color(COLOR_G));
                        break;
                case USER_STATE_REAL_REGISTERED_BLACK:
                        display_set_color(set_yuv_color(COLOR_BK));
                        break;
                default:
                        display_set_color(set_yuv_color(COLOR_R));
                        break;
        }   
# 可以在识别注册、未注册、黑名单中添加对应操作，重新编译覆盖到板子上。       
# 值得注意的是，摄像头的预览并不是用 QT 完成的，是用 Rkfacial，所有画框、
# 信息绘制通过配置 Rkfacial 回调函数实现。等修改完成后就可以直接 make 
# 编译。如果最终修改想固化到 buildroot 中，需要把修改添加到 sdk/app/QFacialGate
```

* 修改完成后通过 adb 部署
```
adb shell "/etc/init.d/S06_QFacialGate stop"
adb push QFacialGate /
adb shell "cp /QFacialGate /usr/bin/QFacialGate"
adb shell "/etc/init.d/S06_QFacialGate start"
```
![](../../../rv1126b_img/AIO-1126BJD4V0/rkfacial.jpg)

* 相关开发文档

QFacialGate
* 路径：/path/to/sdk/app/QFacialGate/doc/Rockchip_Instruction_Linux_QFacialGate_CN.pdf

Rkfacial
* 路径：/path/to/sdk/external/rkfacial/doc/Rockchip_Instruction_Rkfacial_CN.pdf

Web 后端开发框架
* 路径：/path/to/sdk/docs/RV1126_RV1109/ApplicationNote/Rockchip_Developer_Guide_Linux_Application_Framework_CN.pdf

Web 网页端介绍
* 路径：/path/to/sdk/docs/RV1126_RV1109/ApplicationNote/Rockchip_Instructions_Linux_Web_Configuration_CN.pdf