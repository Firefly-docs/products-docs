# Speech Module
## MIC array
### Product Parameters
* Model: MOD-MIC-4XAnalog
* Connection method: USB connection

### Physical map
![](../../../rk3308_img/Core-3308Y/module/module_speech-MOD-MIC-4XAnalog.jpg)

### Digital MIC Instructions for Use
#### Connection Diagram
![](../../../rk3308_img/Core-3308Y/module/module_speech-MOD-MIC-4XAnalog-connect.jpg)
#### Recording

Check the sound card recognized by the system, where `USB-Audio` is the `USB` sound card

````
/ # cat /proc/asound/cards
  0 [rockchiprk3308b]: rockchip_rk3308-rockchip,rk3308b-acodec
                       rockchip,rk3308b-acodec
  1 [Audio ]: USB-Audio - AC108 USB Audio
                       XPowers AND ST AC108 USB Audio at usb-ff440000.usb-1.1, full speed
  7 [Loopback]: Loopback - Loopback
                       Loopback 1
````

recording:

````
arecord -D hw:1,0 -c 8 -r 16000 -f S16_LE test.wav
````

Play:

````
aplay test.wav
````