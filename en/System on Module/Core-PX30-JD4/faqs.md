# FAQ

## Boot fails and restarts repeatedly?

It is probably due to not enough supply current. Please use a power adapter supporting 12V voltage and 2A current.

## Where is the PX30 chip technical manual?

PX30 chip technical manual link: [Baidu cloud (extract code: ry3r)](https://pan.baidu.com/s/1hF4GYq3XcsWO9JddqE0OAA#list/path=%2Fsharelink1414141670-790329647276288%2FPX30%E8%8A%AF%E7%89%87&parentPath=%2Fsharelink1414141670-790329647276288)

## How to burn the MAC address?

You can change MAC address of the AIO-PX30-JD4 by yourself, and you can modify MAC address by the tool -- [WNpctool](https://pan.baidu.com/s/1kU727kF#list/path=%2F) before Connect Device.

## Ubuntu system, if there is no sound after plugging in the headphones, what should I do?

`Menu` -> `Multimedia` -> `PulseAudio Volume Control` -> `Configuration` -> Select a working sound card and turn off another sound card.

## How to capture a log from the board in Android?

`Settings` -> `System` -> `About tablet` -> click 5 times `Build number` -> go back to the previous level `Developer options` -> `Enable logging to save`. After opening the function, the system storage root directory will generate `.LOGSAVE` folder, which includes system logcat and kernel kmsg.