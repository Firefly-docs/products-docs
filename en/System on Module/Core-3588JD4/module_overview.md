# Accessories & Modules

**Core-3588JD4** can be used with the following accessories and modules. Click the links for details. Actual support depends on the specific model.


<CodeBlockTabs defaultValue="camera">
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="camera">Camera Modules</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="camera">
## Camera Modules [step]

Firefly provides MIPI camera modules for image capture and AI vision applications. The tables below are grouped by platform and summarize the cameras (listed by sensor model) whose drivers have been adapted and debugged on each board in the current SDK:

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

<CodeBlockTabs defaultValue="rk3588">  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="rk3588">RK3588</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rk3576">RK3576</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rk356x">RK356X</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rk3506">RK3506</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rv1126">RV1126</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="rv1126b">RV1126B</CodeBlockTabsTrigger>
    <CodeBlockTabsTrigger value="k3">K3</CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="rk3588">

**RK3588**

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

  </CodeBlockTab>

  <CodeBlockTab value="rk3576">

**RK3576**

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| AIO-3576C | √ | — | √ | — | — | — | — |
| AIO-3576JD4 | √ | — | √ | — | — | — | — |
| AIO-3576Q | √ | — | √ | — | — | — | — |
| AIO-3576Q38 | √ | √ | √ | — | — | — | — |
| EXT-ICORE-3576Q38 | √ | — | √ | √ | √ | — | — |
| ROC-RK3576-PC | √ | — | √ | — | — | — | — |

  </CodeBlockTab>

  <CodeBlockTab value="rk356x">

**RK356X**

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| AIO-3566JD4 | √ | — | √ | — | — | — | √ |
| ROC-RK3566-PC | √ | — | √ | — | — | — | √ |
| AIO-3568J | √ | — | √ | — | — | — | √ |
| ITX-3568Q | √ | — | √ | — | — | — | √ |
| ROC-RK3568-PC-SE | √ | — | √ | — | — | — | √ |

  </CodeBlockTab>

  <CodeBlockTab value="rk3506">

**RK3506**

No camera sensors have been confirmed and adapted on this platform yet.

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ROC-RK3506-PC | — | — | — | — | — | — | — |

  </CodeBlockTab>

  <CodeBlockTab value="rv1126">

**RV1126**

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| CAM-C1126S2U | — | — | — | — | — | — | √ |
| C40PL | — | — | — | — | — | √ | — |

  </CodeBlockTab>

  <CodeBlockTab value="rv1126b">

**RV1126B**

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| AIO-1126BJD4 | √ | √ | √ | — | — | √ | — |
| AIO-1126BQ38 | √ | √ | √ | — | — | √ | — |
| CAM-1126BQ38 | √ | — | — | √ | √ | — | — |
| AIO-1126BJD4V0 | √ | — | — | — | — | √ | — |
| AIO-1126BJD4V0 (old carrier board) | √ | — | — | — | — | √ | — |

  </CodeBlockTab>

  <CodeBlockTab value="k3">

**K3**

| Board | [IMX415](../../Accessories/IMX415/started.md) | IMX586 | [XC7160](../../Accessories/CAM-8MS1M/started.md) | SC3336 | IMX335 | OS04A10 | [gc2053/gc2093](../../Accessories/CAM-2MS2MF/started.md) |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| CORE-K3JD4 | — | — | — | — | — | — | — |
| ITX-K3L | √ | — | — | — | — | — | — |
| AIBOX-K3 | — | — | — | — | — | — | — |

  </CodeBlockTab>
</CodeBlockTabs>
  </CodeBlockTab>

</CodeBlockTabs>


