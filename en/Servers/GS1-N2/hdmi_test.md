# HDMI display test

GS1-N2 does not have a graphics card chip, and the HDMI output part of the main control does not use the standard framebuffer driver. When it is shipped from the factory, it is not displayed when it is connected to HDMI.

If the user wants to test the HDMI display, he can connect to the HDMI interface, and then execute the `test-hdmi` script:

```shell
linaro@bm1684:~$ sudo -i
root@bm1684:~# test-hdmi
found (1024, 768) @ 60 fps
entry[0] = (1024, 768) added
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1024, 768) @ 60 fps
found (1920, 1080) detailed timing desc
entry [1] = (1920, 1080) added
found (1920, 1080) detailed timing desc
found (1920, 1080) detailed timing desc
found (1920, 1080) detailed timing desc
```