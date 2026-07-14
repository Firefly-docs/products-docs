# FAQs

### 开机异常并循环重启  

可能是电源电流不够，请使用电压为12V，电流为 2.5A~3A 的电源。

### ubuntu 用户名和密码  

用户：root 密码：firefly
用户：firefly 密码：firefly

### Git 链接地址  
[https://bitbucket.org/T-Firefly/firenow-lollipop](https://bitbucket.org/T-Firefly/firenow-lollipop)
### MAC 地址烧写    
AIO-3128C 的 MAC 地址可以让用户自己更改，请使用 SDK 下的统一动态库工具 RKTools/windows/UpgradeDllTool_v1.35.zip 烧写 MAC 地址。

### Android下如何让系统抓取LOG？
Settings(设置)->About phone(关于手机)->点击5下Build number(版本号)->Developer options(开发者选项)->Enable logging to save(启用日志保存)
打开功能后，系统的storage根目录下就会生成.LOGSAVE文件夹，里面包括系统logcat和内核kmsg。

## 打开Root权限
Android系统有很多很强大的功能都需要用到root权限，开发者经常在使用的时候遇到权限的问题，
那如何在Firefly平台上开启系统的root权限功能呢？Firefly已在系统添加启动root权限的功能，具体的步骤如下：
1. 在Settgins apk里面找到About device然后点击进去
2. 点击Build number 7次后会提示(you are now a developer)
3. 然后返回上一级点击Developer options选项后，在选项中点击Enable ROOT就打开root权限功能
![](../../../rk3128_img/AIO-3128C/android_root.png)

## 网络ADB的使用
adb调试模式有两种：1、使用usb线；2、使用网络。<br />
使用网络adb模式：开发板跟PC端需处于同一局域网内，可以使用有线网的方式，或是让PC端跟开发板连接在同一wifi路由下，亦可通过PC端创建wifi热点让开发板连接都可以。
*  设置->开发者选项->网络ADB调试
![](../../../rk3128_img/AIO-3128C/net_adb.png)<br />


*  用`busybox ifconfig`查看开发板的IP地址，PC端通过网络访问<br />
```
adb connect + IP
adb shell
```
