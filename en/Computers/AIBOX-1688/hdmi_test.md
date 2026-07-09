# HDMI Display Test

AIBOX-1688 supports standard HDMI2.0 output.

You can test the HDMI interface with the following commands:

```shell
insmod /mnt/system/ko/soph_drm.ko # Install display framework driver
systemctl stop SophonHDMI.service # Stop HDMI display interface
systemctl restart SophonHDMI.service  # Restore HDMI display interface
```
You can also test using `modetest`:

```shell
modetest -M cvitek -s 42@40:1920x1080-60@RG24
```
Where `42@40:1920x1080-60@RG24` should be modified according to the actual resolution of the connected monitor.