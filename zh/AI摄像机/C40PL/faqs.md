# FAQs

## SDK 版本
* 查看当前 SDK 版本
```
cd path-to-sdk/
realpath .repo/manifest.xml
```

* 回退 SDK 版本
```
cd path-to-sdk/
# 链接需要回退的 SDK 版本文件名
ln -sf .repo/manifests/rv1126_rv1109/xxx.xml .repo/manifest.xml
# 开始回退
repo sync -c
```

## NPU 版本
* SDK 版本是 rv1126_rv1109_linux_release_20220324_v2.2.5b.xml 版本及日期为 2022.03.24 以上版本 NPU 版本默认为 NPU 1.7.0 。
* 切换 NPU 版本：根据 mk 文件描述的 RK_CFG_BUILDROOT 变量修改 buildroot 相对应的配置文件，添加以下配置
```
BR2_PACKAGE_NPU_ARC=y
```
编译烧录系统后在 RV1126_RV1109 上执行 /npu_arc/usr/bin/npu_arc_env.sh 脚本配置 NPU 1.6.0 环境。该操作的原理是卸载 1.7.0 的内核 NPU ko 模块驱动，然后安装 1.6.0 的内核 NPU ko 模块驱动。

## AI 算法
### ROCKX 算法
Rock-X SDK 是基于 RK3399Pro/RK180X 平台的一套 AI 组件库。开发者通过 Rock-X SDK 提供的 API 接口能够快速构建 AI 应用。目前已适用于 RV1126_RV1109 平台。ROCKX 算法免费，缺点是无法进行接口函数的优化。更多 ROCKX 相关详细内容请查阅 SDK 中的 `sdk/external/rockx/doc/Rockchip_Developer_Guide_RockX_SDK_CN.pdf` 文件。
### 虹软人脸识别算法
RV1126_RV1109 已适配虹软人脸识别算法。仅适用于 NPU 1.6.0 版本，不适用更高的 NPU 版本（含 NPU 1.7.0 版本）。`目前 NPU 1.6.0 以上的版本（不含 NPU 1.6.0）使用虹软人脸识别算法会出现严重的内存泄露`，望周知。虹软人脸算法激活码请联系商务获取。
### 薪火车牌识别算法
RV1126_RV1109 已适配薪火车牌识别算法。官方资源下载界面可下载薪火车牌识别固件。车牌识别应用源码位于 SDK 中的 `sdk/external/xhlpr_app` 文件夹中。车牌识别算法激活码请联系商务获取。
