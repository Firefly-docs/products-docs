# Audio Usage

AIO-8550JD4 has 1 I2S signal for HDMI audio playback and 1 I2S signal for codec audio. (headphone)

## Low-level Playback

Low-level only supports 48K sample rate, 16 bit wav audio.

Need to use agmplay tool:

```bash
# set codec record path
amixer -q set "PGAL Select" "Line 2P"
amixer -q set "PGAR Select" "Line 2N"

# set record volume
amixer -q set "ADCL" "200"
amixer -q set "ADCR" "200"

# record for 5 sec
agmcap -D 100 -d 101 -i MI2S-LPAIF-TX-PRIMARY -dkv 0xA3000004  -c 2 -r 48000 -b 16 -T 5 test.wav

# set playback volume
amixer -q set "DACL" "200"
amixer -q set "DACR" "200"

# play audio through headphone
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-PRIMARY -dkv 0xA2000001

# play audio through HDMI
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

## Upper-level Application

Under development...Coming Soon