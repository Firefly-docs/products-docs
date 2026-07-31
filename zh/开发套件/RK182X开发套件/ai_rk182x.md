# RK1820/RK1828 平台
主控通过 PCIe ⾼速接⼝连接协处理器:
* RK3588 主控（Host） ：作为系统核⼼，负责任务调度、资源分配和整体控制。
* RK1820/RK1828 协处理器（Device） ：作为 AI 计算加速单元，专注于⾼性能神经⽹络推理任务。
* PCIe ⾼速接⼝（通信接⼝） ：实现主控与协处理器之间的低延迟、 ⾼带宽数据交互。

## rknn-smi
rknn-smi (System Management Interface) ⼯具⽤于 RK1820/RK1828 设备信息收集、功能配置、⽇志管理等功能。

* 查询软件版本信息: `sudo rknn-smi -v`
* 查询硬件版本信息: `sudo rknn-smi info -l`
* 状态监控: `sudo rknn-smi info -w`
* 性能模式: `sudo rknn-smi set -t work_mode -s 2`
* 查询功耗：此功能硬件上不支持
    * `sudo rknn-smi info -t power`

具体使用，详见 RK182X SDK 里的 `docs/Tools/Rockchip_User_Guide_RKNN-SMI_Tool_CN.pdf`

### 概率性 "Failed to initialize rknnsmi"
`/lib/systemd/system/rknn3.service` 适当添加延迟：

```
[Unit]
Description=rknn3 runtime service
DefaultDependencies=no
After=local-fs.target

[Service]
Type=forking
ExecStartPre=/bin/sleep 3 # 延迟 3 秒
ExecStart=/bin/rknn3_startup start
ExecStop=/bin/rknn3_startup stop

[Install]
WantedBy=sysinit.target
```

## RKNN3
RK182X SDK 的 rknn 目录：
```
rknn/
├── rknn3-model-zoo
├── rknn3-runtime
├── rknn3-toolkit
└── rknn-gstreamer-plugins
```

**RKNN3 SDK 框图**
![](../../../gs1-n2_img/AIO-GS1N2-RK182X/RKNN3-SDK-Block-Diagram.png)

### RKNN3 Model Zoo
提供 RK1820/RK1828 平台上经典模型的部署示例。
<br>
[github](https://github.com/airockchip/rknn3-model-zoo)
### RKNN3 Runtime
RKNN3 C API 是 RKNN3 Runtime（运⾏时库）的 C 语⾔接口。
<br>
开发者使⽤ C/C++ 开发应⽤程序，通过 RKNN3 C API 部署模型推理。
### RKNN3 Toolkit
RKNN3 Toolkit 是为用户提供在 PC 平台上进行模型转换、推理和性能评估的开发套件。
<br>
**RKNN3 Toolkit** 与 [RKNN-Toolkit](https://github.com/airockchip/rknn-toolkit) 和 [RKNN-Toolkit2](https://github.com/airockchip/rknn-toolkit2) **不兼容**。
<br>
[github](https://github.com/airockchip/rknn3-toolkit)