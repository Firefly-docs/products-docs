# Accessories & Modules

**Core-3588L** can be used with the following accessories and modules. Click the links for details. Actual support depends on the specific model.


<CodeBlockTabs defaultValue="camera">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="camera">Camera Modules</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="display">Display Modules</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="camera">
## Camera Modules [step]

Firefly provides MIPI camera modules for image capture and AI vision applications. The table below summarizes the cameras (listed by sensor model) whose drivers have been adapted and debugged on each board in the current SDK:

> In the table, `√` only means the driver has been adapted and debugged under the current SDK device tree, **and does not mean IQ (image quality) has been tuned**. Actual image results such as auto exposure, white balance and noise reduction depend on the corresponding ISP/IQ configuration and must be confirmed with the SDK in use.
>
> The following is primarily based on Linux systems. For Android systems, please confirm based on the device tree configuration of the actual SDK.
>
> `—` means the SDK has not completed adaptation verification, which does not necessarily mean the hardware is incompatible.

Only the following three cameras are for sale. The sensor-to-product mapping is as follows:

- IMX415 → [800W CAM-8MS1M (IMX415)](../../Accessories/IMX415/started.md)
- XC7160 → [800W single-lens camera CAM-8MS1M](../../Accessories/CAM-8MS1M/started.md)
- gc2053/gc2093 → [2MP dual-lens camera CAM-2MS2MF](../../Accessories/CAM-2MS2MF/started.md)

Other sensors/cameras are only covered by adaptation notes and are not sold by Firefly. If you need to purchase one, please source the camera and its matching FPC cable yourself.

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| AIO-3588Q | √ | √ | √ | — | — | — | — |
| ITX-3588 | √ | √ | √ | — | — | — | — |
| AIO-3588L | √ | √ | √ | — | — | — | — |
| AIO-3588JD4 | √ | — | — | — | — | — | — |
| AIO-3588SG | √ | — | √ | — | — | — | — |
| AIO-3588SJD4-AI | √ | — | — | — | — | — | — |
| AIO-3588SJD4 | √ | — | √ | — | — | — | — |
| ROC-3588S-PC | √ | √ | √ | — | — | — | √ |
| AIO-3576C | √ | — | √ | — | — | — | — |
| AIO-3576JD4 | √ | — | √ | — | — | — | — |
| AIO-3576Q | √ | — | √ | — | — | — | — |
| AIO-3576Q38 | √ | √ | √ | — | — | — | — |
| EXT-ICORE-3576Q38 | √ | — | √ | √ | √ | — | — |
| ROC-RK3576-PC | √ | — | √ | — | — | — | — |
| AIO-1126BJD4 | √ | √ | √ | — | — | √ | — |
| AIO-1126BQ38 | √ | √ | √ | — | — | √ | — |
| CAM-1126BQ38 | √ | — | — | √ | √ | — | — |
| AIO-1126BJD4V0 | √ | — | — | — | — | √ | — |
| ROC-RK3506-PC | — | — | — | — | — | — | — |
| AIO-3566JD4 | √ | — | √ | — | — | — | √ |
| ROC-RK3566-PC | √ | — | √ | — | — | — | √ |
| AIO-3568J | √ | — | √ | — | — | — | √ |
| ITX-3568Q | √ | — | √ | — | — | — | √ |
| ROC-RK3568-PC-SE | √ | — | √ | — | — | — | √ |
| AIO-1126BJD4V0 (old carrier board) | √ | — | — | — | — | √ | — |
| CAM-C1126S2U | — | — | — | — | — | — | √ |
| C40PL | — | — | — | — | — | √ | — |
| CORE-K3JD4 | — | — | — | — | — | — | — |
| ITX-K3L | √ | — | — | — | — | — | — |
| AIBOX-K3 | — | — | — | — | — | — | — |
  </CodeBlockTab>

  <CodeBlockTab value="display">
## Display Modules [step]

Firefly has adapted the following displays. Only the **DM-M10R800-V3S** (10.1" 800×1280 MIPI screen) is for sale; the screens corresponding to all other columns in the table are not sold and only provide corresponding device tree (DTS) configurations as reference, so please prepare the screen yourself if needed.

> The following is primarily based on Linux systems. For Android systems, please confirm based on the device tree configuration of the actual SDK.
>
> In the table, `√` means the current SDK device tree has completed adaptation and debug verification; `—` means the SDK has not completed adaptation verification, which does not necessarily mean the hardware is incompatible.

Adapted screen models:

- **DM-M10R800-V3S**: 10.1" / 800×1280 / MIPI, for sale
- NE156QUM-N66: 15.6" / 4K / EDP, not sold (DTS reference only)
- 156FHM-T06: 15.6" / 1080P / EDP, not sold (DTS reference only)
- 10.1" 1080P dual-channel LVDS screen: not sold (DTS reference only)
- 10.1" 800×1280 single-channel LVDS screen: not sold (DTS reference only)

| Board | [10.1" 800×1280 MIPI](../../Accessories/DM-M10R800-V3S/started.md) | 15.6" 4K EDP | 15.6" 1080P EDP | 10.1" 1080P LVDS | 10.1" 800×1280 LVDS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| AIO-3588Q | √ | — | √ | — | — |
| ITX-3588 | √ | — | — | — | — |
| AIO-3588L | √ | — | — | — | — |
| AIO-3588JD4 | — | — | — | — | — |
| AIO-3588SG | √ | √ | — | — | — |
| AIO-3588SJD4-AI | — | — | — | — | — |
| AIO-3588SJD4 | √ | — | — | — | — |
| ROC-3588S-PC | √ | — | — | — | — |
| AIO-3576C | √ | √ | — | √ | √ |
| AIO-3576JD4 | — | — | — | — | — |
| AIO-3576Q | √ | — | — | — | — |
| AIO-3576Q38 | — | — | — | — | — |
| EXT-ICORE-3576Q38 | √ | — | — | — | — |
| ROC-RK3576-PC | √ | — | — | — | — |
| AIO-1126BJD4 | √ | — | — | — | — |
| AIO-1126BQ38 | √ | — | — | — | — |
| CAM-1126BQ38 | — | — | — | — | — |
| AIO-1126BJD4V0 | √ | — | — | — | — |
| ROC-RK3506-PC | √ | — | — | — | — |
| AIO-3566JD4 | √ | — | √ | — | — |
| ROC-RK3566-PC | √ | — | — | — | — |
| AIO-3568J | √ | — | √ | — | — |
| ITX-3568Q | √ | — | — | — | √ |
| ROC-RK3568-PC-SE | √ | — | √ | — | — |
| AIO-1126BJD4V0 (old carrier board) | √ | — | — | — | — |
| CAM-C1126S2U | √ | — | — | — | — |
| C40PL | √ | — | — | — | — |
| CORE-K3JD4 | — | — | — | — | — |
| ITX-K3L | √ | — | — | — | — |
| AIBOX-K3 | — | — | — | — | — |
  </CodeBlockTab>
</CodeBlockTabs>


