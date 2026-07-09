# Audio Usage

AIBOX-8550 has 1 I2S signal for HDMI audio playback.

## Low-level Playback

Low-level only supports 48K sample rate, 16 bit wav audio.

Need to use agmplay tool:

```bash
# play audio to HDMI
agmplay test.wav -D 100 -d 100 -i MI2S-LPAIF-RX-SECONDARY -dkv 0xA2000004
```

## Upper-level Application

Under development...Coming Soon