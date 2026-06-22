## 编译 Thunder-boot 固件
目前 CORE-1126-JD4 最新的 SDK 已适配快速启动。编译快速启动固件选项：
```
# 编译 CORE-1126-JD4 V1.1 快速启动固件 适配的屏幕为 DM-M10R800 V2 屏幕模组
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-jd4-BE-45-tb-v11.mk
```
快速启动固件启动后，MIPI DM-M10R800 V2 屏幕模组可预览摄像头画面。若接入网络，还可预览摄像头推流 rtsp。
* Linux PC 预览命令：
```
vlc rtsp://<设备 ip 地址>
```
* Windows PC 则需要下载 vlc 应用，并且输入串流地址来预览。

![](/img/CORE-1126BJD4/windows_vlc.jpg)
注意：该快速启动固件`仅适用于 CORE-1126-JD4 V1.1 版本的核心板`。核心板版本详见硬件上的丝印。