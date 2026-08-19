## Sound Card

### EarPhone && Speak
Both EarPhone and Speak are dual-channel interfaces

Use the aplay command to play audio in wav format
```
#path-to indicates the absolute path to store the audio
aplay /path-to/audio-name.wav
```
### Mic
* Mic is off by default. You need to turn it on to use it.
```
#The command sets the default recording sound card channel.
amixer cset numid=2,iface=MIXER,name='Capture MIC Path' 1

#Set the default recording sound card channel.
alsamixer
#Set the Capture MIC Path to Main MIC
```
* Mic records audio
```
arecord -l											#View all available MIC devices.
arecord -Dhw:0,0 -f cd -d 10 /path-to/audio.wav	#Select the sound card and record audio.
#Note: Mic only supports mono recording
```