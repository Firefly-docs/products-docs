# Technical Case


Due to its compact size, rich expansion functions and high cost performance, `CAM-C1126S2U/CAM-C1109S2U` has a wide range of application scenarios. Such as: face recognition gate, smart door lock, gesture recognition, smart travel, etc. Here are some mainstream application scenarios of `CAM-C1126S2U/CAM-C1109S2U`.

## Face Recognition Gate

The application scenarios of face recognition gate are divided into two cases: external screen and bare metal. The two application scenarios are compatible with each other, and no software or hardware modification is required. Before use, you only need to select and adapt the face recognition algorithm that suits you. The following takes the adapted ArcSoft algorithm as an example to introduce the application scenarios of face recognition gate. PS: The official gate firmware corresponds to this scenario.

### External Screen

The external screen can give users a complete and good user experience. The screen supports touch, and more operations can be completed directly on the screen after building a complete interactive interface. The following is a real machine display.

* Adapt high-quality `RGB` camera with professional ISP image quality debugging. The following is the recognition preview effect.

![](../../../rv1126_img/CAM-C1109S2U/pass.jpg)

* Adapt `IR` camera to effectively prevent cheating such as photos, videos and wax figures.

![](../../../rv1126_img/CAM-C1109S2U/npass.jpg)

* `TYPE-C OTG` provides hardware support for control background, face registration management, and gate background maintenance.

![](../../../rv1126_img/CAM-C1109S2U/weblogin.png)

### Bare Metal Gate

Bare metal means no external screen. Except that the bare metal cannot interact with the user through the screen, all functions are the same. You can even modify the debug serial port to a normal serial port to achieve the function of face recognition and result output without connecting any device. In addition, the face library can still be imported into the device through `TYPE-C OTG`. This scenario can minimize the cost of face recognition gate.

## AI_UVC Camera

`CAM-C1126S2U/CAM-C1109S2U` can virtualize itself into UVC and RNDIS devices, and can be connected to any host computer through the TYPE-C OTG interface to make an AI_UVC camera. The following is a Windows platform application using ArcSoft
algorithm. PS: The official UVC firmware corresponds to this scenario.

* Face recognition effect.

![](../../../rv1126_img/CAM-C1109S2U/arcuvc_face.png)

## AI Compute Stick

`CAM-C1126S2U/CAM-C1109S2U` motherboard itself can also be used as an AI compute stick, which can assist the host computer in performing some AI calculations by connecting to the host computer. The device uses `TYPE-C OTG` as a slave device to access the host, and calls `npu` to assist in calculations through the official `RKNN-Toolkit`. See the [《NPU》](usage_npu.md) section for details.
