# Audio Usage

Firefly-Q6490A uses an ES8390 as the sound card. Both the output channels and the input channels have two routes.

* INPUT1 <- Headphone jack recording
* INPUT2 <- Expansion Pin Header, for external line in
* OUTPUT1 -> Expansion Pin Header, for external line out
* OUTPUT2 -> Headphone jack playback

The sound service used by the system is ALSA + Pipewire

## Playback
```bash
# You can set the volume first
# DACL and DACR are the volume of the left channel and the right channel respectively, the value range is 0~255
amixer -q -D hw:QCM6490IDP set DACL 200
amixer -q -D hw:QCM6490IDP set DACR 200

# Play
pw-play test.wav
pw-play test.mp3
```

## Recording
```bash
# Switch the input channel to channel 1
amixer -q -D hw:QCM6490IDP set "PGAL Select" "Line 1P"
amixer -q -D hw:QCM6490IDP set "PGAR Select" "Line 1N"

# Set the recording gain, the value range is 0~255
amixer -q -D hw:QCM6490IDP set ADCL 200
amixer -q -D hw:QCM6490IDP set ADCR 200

# Start recording
pw-record output.wav

# Play the recording
pw-play output.wav
```