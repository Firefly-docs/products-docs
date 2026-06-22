# Display

* The hardware connection between the baseboard and the MIPI screen is shown in the figure:

![](../../../rv1126b_img/AIO-1126BJD4V0/mipi_screen.jpg)

Debian firmware boots into the XFCE desktop environment by default. Since the RV1126B chip lacks a GPU for hardware-accelerated rendering, the desktop environment uses the CPU for rendering. Therefore, under heavy workloads, display stuttering may occur.

* QT application screen redirection

```
#Specify the QT output platform and make the screen rotate 90° clockwise
export QT_QPA_PLATFORM=linuxfb:rotation=90
```
* If the screen is distorted or does not display. Please check whether there is a problem with the LCD cable. Replacing the short cable with high-quality material can solve the distorted screen problem.