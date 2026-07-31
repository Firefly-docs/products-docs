# 接口
## 外壳
![](../../../gs1-n2_img/GS1-N2/interface-enclosure-zh.png)

## 主板
![](../../../gs1-n2_img/GS1-N2/interface-mainboard-zh.png)

|主模组|仅安装主模组时|
|----|----|
|RK3588(Core-3588JD4)|1. SATA0、PCIe0、PCIe WiFi(M.2) ：三选一<br>2. PCIe1(PCIe 3.0)、连接副核心板：二选一<br>3. SATA2 可用（主模组 Core-3588JD4 专用，为其它模组时 SATA2 不可用）<br>4. 其他 PCIe / SATA 均不可用|
|RK3576(Core-3576JD4)|1. SATA0、PCIe0、PCIe WiFi(M.2) ：三选一<br>2. SATA1、PCIe1：二选一<br>3. 其他 PCIe / SATA 均不可用|
|SOPHGO(Core-1688JD4)|1. SATA0、PCIe0、PCIe WiFi(M.2)：三选一<br>2. PCIe1：可用<br>3. 其他 PCIe / SATA 均不可用|
|NVIDIA(Jetson Orin Nano)<br>NVIDIA(Jetson Orin NX)|1. PCIe0、PCIe WiFi(M.2) ：二选一<br>2. PCIe1：可用<br>3. TF卡、管理网口：不可用<br>4. 其他 PCIe / SATA 均不可用|

|副模组|仅安装副模组时|
|----|----|
|1. 可选主模组里任意一款<br>2. 副模组可为空|1. RES_PCIe 均可用<br>2. 副模组为以下模组时，PCIe2才可用：<br>&ensp;&ensp;Core-3588JD4、Jetson Orin Nano、Jetson Orin NX|