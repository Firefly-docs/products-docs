# Audio
AIBOX-186 has two audio outputs and one audio input.

## Audio Output

Users can output audio through the headphone jack and HDMI port.

### Interface Switching

Switch between the headphone and HDMI interfaces in the system settings, selecting one for output:

![](../../../bm1688_img/AIBOX-186/Sound.png)

### Command Line Mode

In the terminal, execute the command:

```shell
# cat /proc/asound/cards
 0 [HDA            ]: tegra-hda - NVIDIA Jetson Orin Nano HDA
                      NVIDIA Jetson Orin Nano HDA at 0x3518000 irq 120
 1 [APE            ]: tegra-ape - NVIDIA Jetson Orin Nano APE
                      NVIDIA-NVIDIAJetsonOrinNanoDeveloperKit-NotSpecified-Jetson
```

* `HDA` represents the HDMI sound card, device number `3`
* `APE` represents the headphone sound card, device number `0`

* HDMI 

    ```shell
    aplay -D hw:HDA,3 hdmi.wav # where hdmi.wav needs to be a stereo format file
    ```

* Headphones

    ```shell
    amixer -c APE cset name="I2S2 Mux" ADMAIF1
    aplay -D hw:APE,0 test.wav
    ```

## Audio Input

Using the `APE` sound card, connect the headphones to start recording, execute the command line:

```shell
amixer -c APE cset name="ADMAIF1 Mux" I2S2
arecord -D hw:APE,0 -c 2 -r 16000  -f S32_LE test.wav
```