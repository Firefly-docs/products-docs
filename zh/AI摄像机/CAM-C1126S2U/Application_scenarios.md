`CORE-1126-JD4/CORE-1109-JD4` 金手指保留了 Soc 的大部分接口，用户可以根据自身需要自己制作底板来满足各种应用场景。下面
介绍一下配到官方 `MB-1126-JD4` 底板下的几种应用场景。

## AI 网络摄像头

底板拥有`以太网接口`、`MIPI-CSI`、`POE` 等硬件基础，可以实现 AI 智能网络摄像头。在设备上可以实现 AI 识别和 RTSP 推流，
在 WEB 端或者 RTSP 播放器就可以查看到预览画面。

![](../../../rv1126_img/CAM-C1126S2U/web.png)

## 人脸识别闸机

设备预留了两组 `MIPI-CSI` 接口，可以适配 RGB & IR 双目摄像头来满足人脸识别闸机的需求。

![](../../../rv1126_img/CAM-C1126S2U/facial_gate.jpg)

人脸识别闸机 Web 管理界面

![](../../../rv1126_img/CAM-C1126S2U/facial_gate_web.png)

## 集群边缘计算

`CORE-1126-JD4/CORE-1109-JD4` 除了可以适配 `MB-1126-JD4` 单独使用外，还适配了官方的 CS-R1/CS-R2 系列集群服务器。关于
集群服务器相关 WIKI（[CS-R1](http://wiki.t-firefly.com/zh_CN/Cluster-Server-R1/started.html)/[CS-R2](http://wiki.t-firefly.com/zh_CN/Cluster-Server-R2/Firefly_Cluster_server_R2_product_documentation.html)）。

* CS-R1
![](../../../rv1126_img/CAM-C1126S2U/CS-R1.png)

* CS-R2
![](../../../rv1126_img/CAM-C1126S2U/CS-R2.png)

