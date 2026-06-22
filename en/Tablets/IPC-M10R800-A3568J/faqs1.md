## How to adjust HDMI output resolution manually ?

In addition to adjusting the resolution in `Setting->Display->HDMI->Resolution`, we could also use command below to switch 4K resoultion and flash screen:

```
# 4k60:
setprop persist.sys.resolution.main 3840x2160@60 
# Set sys.display.timeline base on the default num to plus one for updating resolution
setprop sys.display.timeline 1
```
## The 3.5mm headphone jack is connected to the national standard headphone abnormal?

At present, IPC-M10R800-A3568J only supports the American standard type of headphones(CTIA). For the national standard type of headphones(OMTP), the hardware is not compatible, and there will be the phenomenon of dual sound channels in both the left and right channels.
