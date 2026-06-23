# RK1820/RK1828 

* RK3588 Soc(Host) : As the core of the system, responsible for task scheduling, resource allocation, and overall control.
* RK1820/RK1828 Coprocessor(Device) : As an AI computing acceleration unit, we focus on high-performance neural network inference tasks.
* PCIe : Realize low latency and high bandwidth data exchange between the main control and co processors.

## rknn-smi
rknn-smi (System Management Interface) use as RK1820/RK1828 equipment information collection, function configuration, and log management functions.

* Software Version: `sudo rknn-smi -v`
* Hardware Version: `sudo rknn-smi info -l`
* Status: `sudo rknn-smi info -w`
* Performance: `sudo rknn-smi set -t work_mode -s 2`
* Power Consumption: This feature is not supported on hardware
    * `sudo rknn-smi info -t power` 

How to use, see RK182X SDK: `docs/Tools/Rockchip_User_Guide_RKNN-SMI_Tool_EN.pdf`

### Probability "Failed to initialize rknnsmi"
`/lib/systemd/system/rknn3.service` add appropriate delay:

```
[Unit]
Description=rknn3 runtime service
DefaultDependencies=no
After=local-fs.target

[Service]
Type=forking
ExecStartPre=/bin/sleep 3 # delay 3 seconds
ExecStart=/bin/rknn3_startup start
ExecStop=/bin/rknn3_startup stop

[Install]
WantedBy=sysinit.target
```

## RKNN3
RK182X SDK dictionary:
```
rknn/
├── rknn3-model-zoo
├── rknn3-runtime
├── rknn3-toolkit
└── rknn-gstreamer-plugins
```

**RKNN3 SDK Block Diagram**
![](../../../aibox_img/AIBOX-K3/RKNN3-SDK-Block-Diagram.png)

### RKNN3 Model Zoo
Provide deployment examples of classic models on the RK1820/RK1828 platform.
<br>
[github](https://github.com/airockchip/rknn3-model-zoo)
### RKNN3 Runtime
The RKNN3 C API is the C language interface for the RKNN3 Runtime.
<br>
Developers use C/C++ to develop application programs and deploy model inference through RKNN3 C API.
### RKNN3 Toolkit
RKNN3 Toolkit is a development kit that provides users with model transformation, inference, and performance evaluation on the PC platform.
<br>
**RKNN3 Toolkit** different from [RKNN-Toolkit](https://github.com/airockchip/rknn-toolkit) and [RKNN-Toolkit2](https://github.com/airockchip/rknn-toolkit2).
<br>
[github](https://github.com/airockchip/rknn3-toolkit)