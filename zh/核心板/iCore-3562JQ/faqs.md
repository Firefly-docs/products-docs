# FAQs

## 修改 kernel/buildroot 等配置后编译烧录发现不生效
通过`make menuconfig`修改配置之后，使用`build.sh`编译，烧录发现修改没有生效。然后检查 config 发现之前的修改消失了。

这是因为只将修改保存到了临时文件`.config`，编译时`build.sh`会用配置文件覆盖掉`.config`

所以修改后要保存到配置文件：
```bash
make ARCH=arm64 savedefconfig
mv defconfig arch/arm64/configs/firefly_linux_defconfig
```
之后再使用`build.sh`编译。


## 开机异常并循环重启怎么办？

有可能是电源电流不够，请确认电源适配器功率足够。

## Ubuntu 默认的用户名和密码是什么？

* 用户名：`firefly`
* 密码：`firefly`
* 切换超级用户 `sudo -s`

## Android 下如何让系统抓取 LOG？

1、`Settings(设置)` -> `About tablet(关于平板电脑)` -> 点击7下 `Build number(版本号)` 

2、`Settings(设置)`->`System(系统)`->`Advanced(高级)`->`Developer options(开发者选项)` -> `Android LogSave(日志采集器)`。打开功能后，系统的 `/data/vendor/logs` 目录下就会生成 对应日期的日志 文件夹，里面包括系统 logcat 和内核 kmsg。