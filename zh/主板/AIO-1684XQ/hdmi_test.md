# HDMI 显示测试

AIO-1684XQ 不带有显卡芯片，且主控 HDMI 输出部分并没有使用标准的 framebuffer 驱动，出厂情况下接入 HDMI 是没有显示的。

若用户要测试 HDMI 显示可接入  HDMI 接口，然后执行 `test-hdmi` 脚本：

```shell
linaro@bm1684:~$ sudo -i
root@bm1684:~# test-hdmi 
found (1024, 768) @ 60 fps
entry [0] = (1024, 768) added
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1920, 1080) detailed timing desc
entry [1] = (1920, 1080) added
found (1920, 1080) detailed timing desc
found (1920, 1080) detailed timing desc
found (1920, 1080) detailed timing desc
```