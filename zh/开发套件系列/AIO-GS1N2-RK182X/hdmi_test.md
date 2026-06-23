# HDMI 显示测试

RK182X开发套件  支持标准的 HDMI2.0 输出。

可以通过以下命令去测试 HDMI 接口：

```shell
insmod /mnt/system/ko/soph_drm.ko # 安装显示框架驱动
systemctl stop SophonHDMI.service # 关闭 HDMI 的显示界面
systemctl restart SophonHDMI.service  # 恢复 HDMI 的显示界面
```
也可以通过 `modetest` 去测试：

```shell
modetest -M cvitek -s 42@40:1920x1080-60@RG24
```
其中 `37@35:1920x1080-60@RG24` 根据你实际接的显示器以及最终生成的`crtc ` 去填写。 