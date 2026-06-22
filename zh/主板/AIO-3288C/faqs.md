# FAQ

## 开机异常并循环重启


可能是电源电流不够，请使用电压为 12V，电流为 2.5A~3A 的电源。


## Ubuntu 用户名和密码

```bash
用户：root 密码：firefly
用户：firefly 密码：firefly
```

## Git链接地址

[https://bitbucket.org/T-Firefly/firefly-rk3288](https://bitbucket.org/T-Firefly/firefly-rk3288)

## MAC 地址和序列号烧写

AIO-3288C 的 MAC 地址可以让用户自己更改，请使用 SDK 下的统一动态库工具 `RKTools/UpgradeDllTool_Release_v1.3.tar.gz` 烧写 MAC 地址和序列号

## 打开 Root 权限

Android 系统有很多很强大的功能都需要用到 root 权限，开发者经常在使用的时候遇到权限的问题。

那如何在 Firefly 平台上开启系统的 root 权限功能呢？Firefly 已在系统添加启动 root 权限的功能，具体的步骤如下：

1. 在 Settgins apk 里面找到 About device 然后点击进去
2. 点击 Build number 7次后会提示(you are now a developer)
3. 然后返回上一级点击 Developer options 选项后，在选项中点击 Enable ROOT 就打开 root 权限功能

![](../../../rk3288_img/faqs_android_root.png)




## OTG 口无法使用 USB 设备

AIO-3288C 有一个双层 USB 接口，底下的接口为 OTG 接口，使用 ADB 时需要在设置->USB->连接PC打钩，如果使用其他 USB 设备，则需要去掉打钩，不然会导致 USB 设备不能使用。

