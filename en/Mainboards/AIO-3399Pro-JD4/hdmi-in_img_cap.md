## HDMI_IN Image capture

**Support AIO-3399J/AIO-3288J**
* Need to solder HDMI_IN input interface
* This function is turned off by default, and the following operations need to be performed in the kernel warehouse:```git revert 8998afb927200ad42b27beacc22cf1e86bfca442```
* Make sure this patch exists:
        [firefly: AIO-3399J: enable HDMI-IN function: The HDMI-IN function is enabled, but other camera and audio functions may not be used normally.](https://gitlab.com/firefly-linux/kernel/-/commit/30e174dc173fba89c9b84a2651ad46cbc68bb0ac)

**Test script**

* test_hdmiin-3399.sh

```
#!/bin/bash

export DISPLAY=:0
export XAUTHORITY=/home/firefly/.Xauthority
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
export LD_LIBRARY_PATH=/usr/lib/gstreamer-1.0

WIDTH=1920
HEIGHT=1080
SINK=kmssink
#SINK=gtksink

echo autospawn = no > /home/firefly/.config/pulse/client.conf
pkill pulseaudio
sleep 1
amixer -c 1 sset 'IN1 Boost' 0
amixer -c 1 sset "RECMIXL BST1" on
amixer -c 1 sset "RECMIXR BST1" on
amixer -c 1 sset 'Mono ADC MIXR ADC1' on
amixer -c 1 sset 'Mono ADC MIXL ADC1' on

gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=${WIDTH},height=${HEIGHT}, framerate=30/1 ! videoconvert ! $SINK & # &>/dev/null &

GST_PID=$!

sleep 2
alsaloop -P hw:1,0 -C hw:1,0 -c 2 -t 500000 &
ALSA_PID=$!

trap '{ kill -9 $GST_PID; kill -9 $ALSA_PID; }' EXIT INT TERM

wait

```