# Camera 使用


无论是 Facial_Gate 和 AI UVC 固件，开机应用都会占用摄像头节点。在测试摄像头之前，请尝试执行以下命令将开机应用关闭。

```bash
[root@RV1126_RV1109:/]# RkLunch-stop.sh
```

摄像头的具体使用可以参考 `/path/to/sdk/external/rkmedia/examples/rkmedia_vi_double_cameras_test.c` 示例代码进行开发。

如果要接 V2 MIPI 屏幕或者 V3 MIPI 屏幕测试双目摄像头显示，需要设置显示分辨率为 1280x800 。


