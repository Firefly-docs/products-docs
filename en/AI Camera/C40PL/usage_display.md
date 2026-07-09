# Display

* The hardware connection between the baseboard and the MIPI screen is shown in the figure:

![](../../../rv1126_img/C40PL/mipi_screen.jpg)

* **Note: The default public version firmware only displays the firefly LOGO icon when the baseboard is not connected to the camera. If the baseboard is connected to the 0S04A10 camera, the screen displays the camera preview interface. The rv1126 system is not equipped with a desktop environment. **

* Screen rotation

```
#Specify the QT output platform and make the screen rotate 90° clockwise
export QT_QPA_PLATFORM=linuxfb:rotation=90
```
* If the screen is distorted or does not display. Please check whether there is a problem with the LCD cable. Replacing the short cable with high-quality material can solve the distorted screen problem.