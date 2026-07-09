# 介绍
## 产品简介

IPC-M10R800-A3568J 工业级行业平板，基于 AIO-3568J 高性能开源平台，采用电容式 10点
触摸屏，全视角高清显示，配置工业级外壳，防尘防干扰，长时间稳定运行，支持 4K 硬解，丰富的接口方便连接各种工业设备。
搭载 ARM 全新 Cortex-A55 架构、四核64位高性能处理器，主频高达 2.0GHz，ARM G52 2EE 图形处理器。RKNN NPU AI 加速器 ( SoC集成 ) 。

![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_info01.png)
![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_info02.png)
![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_info03.png)
![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_info04.png)

## 产品参数
![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_zh_args.png)

## SATA安装
![](../../../rk356x_img/IPC-M10R800-A3568J/ipc-m10r800-a3568j_sata.png)

## 产品资源

* [[Wiki]](https://wiki.t-firefly.com/Core-3568J/started.html) 
包含 Android&Ubuntu 驱动开发等资料(参考 AIO-3568J Wiki)

* [[SDK 下载地址]](https://www.t-firefly.com/doc/download/124.html#other_449) 
Android11.0 SDK 源码

* [[固件 下载地址]](https://www.t-firefly.com/doc/download/124.html#other_488) 
Android11.0 固件

* [[技术交流论坛]](http://dev.t-firefly.com/forum.php)
超过10万企业客户和用户沟通交流平台

## 产品编译方式
### 一键编译
```
./FFTools/make.sh -d rk3568-firefly-aioj-ipc-mipi_M101014_BE45_A1 -j8 -l rk3568_firefly_aioj_ipc-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3568_firefly_aioj_ipc-userdebug
```

## 产品技术支持
IPC-M10R800-A3568J 可以在多种场景实现客户不同方面的需要，在游戏设备，广告机，自动售货机，机器人等
已经广泛的使用，品质和性能在行业内已经有非常好的口碑，专业的技术团队为广大客户解决硬件设计和软件功能上
的各种各样问题。专业技术支持和更详细资料请联系商务。

### 技术案例
* Android 多路编解码
* 双目摄像头数据采集
* Android 双屏异显
* 人脸识别

### 联系方式
* 邮箱：sales@t-firefly.com
* 手机：(+86) 186 8811 7175
* 座机：0760-89881218
* 全国服务热线：4001-511-533
* 地址：广东省中山市东区中山四路 57 号宏宇大厦 2101 室
 <a id="firmware-format"></a>