## HDMI can't display 4K?

Automatically switching to 4K is temporarily not supported. The default display is 1080P. Support will be updated in the future.After setting the 4K resolution manually, you can use the following command to replug the HDMI cable:

```
# 4k60 setting：
setprop persist.sys.resolution.main 3840x2160@60
# Update sys.display.timeline every time you set it (add 1 each time) for resolution to take effect
setprop sys.display.timeline 1
```

## The 3.5mm headphone jack is connected to the national standard headphone abnormal?

At present, EC-R3568PC only supports the American standard type of headphones(CTIA). For the national standard type of headphones(OMTP), the hardware is not compatible, and there will be the phenomenon of dual sound channels in both the left and right channels.

