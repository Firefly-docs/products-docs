## Sound Card

### Mic
* Record audio

```bash
# View all available MIC devices
$ arecord -l

# Select the sound card and record audio
$ arecord -f cd -Dhw:0,0 -d 10 -c 8 /tmp/test.wav
```

PC uses ADB tool to copy the recorded audio to the computer for playback verification

```bash
adb pull /tmp/test.wav ./