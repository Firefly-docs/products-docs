# Audio 
## Output
Users can output audio through the headphone jack and HDMI port.

### Interface Switching

Switch between the headphone and HDMI interfaces in the system settings, selecting one for output.

### 命令行模式
In the terminal, execute the command: `cat /proc/asound/cards`:

* `HDA` represents the HDMI sound card
* `APE` represents the headphone sound card

#### HDMI 
```shell
aplay -D hw:HDA,3 hdmi.wav
```

#### Headphones
```shell
aplay -D hw:APE,0 test.wav
```

## Input
Using the `APE` sound card, connect the headphones to start recording, execute the command line:

```shell
arecord -D hw:APE,0 -c 2 -r 16000  -f S32_LE test.wav
```

