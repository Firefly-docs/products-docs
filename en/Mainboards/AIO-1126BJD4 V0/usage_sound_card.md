## Sound Card

### EarPhone && Speak
Both EarPhone and Speak are dual-channel interfaces

Use the aplay command to play audio in wav format
```
#path-to indicates the absolute path to store the audio
aplay /path-to/audio-name.wav
```
### Mic

* The Mic uses the chip's ADC1 peripheral. It is disabled by default and needs to be enabled using the following steps.

```bash
amixer -c 0 cset name='ACodec ADC Switch' 1						# Enable ADC1
amixer -c 0 cset name='ACodec Digital Gain Volume' 0			# Set the digital volume of ADC1 to maximum. 0 is maximum, 127 is minimum
amixer -c 0 cset name='ACodec PGA Gain Volume' 31				# Set the digital volume of ADC1 to maximum

amixer -c 0 cset name='SAI2 Receive PATH0 Source Select' 1
amixer -c 0 cset name='SAI2 Receive PATH1 Source Select' 0
```
* Mic records audio
```
arecord -l											#View all available MIC devices.
arecord -Dhw:0,0 -f cd -d 10 /path-to/audio.wav	#Select the sound card and record audio.
#Note: Mic only supports mono recording
```

For more detailed configuration information about the RV1126B acodec, please download the "RV1126B acodec Configuration Document" (https://www.t-firefly.com/doc/download/333.html#other_932) for reference.