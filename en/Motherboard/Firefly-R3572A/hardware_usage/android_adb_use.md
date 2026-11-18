# ADB Usage

## Introduction [step]

ADB (Android Debug Bridge) is an Android command-line debugging tool for viewing system logs, transferring files, installing applications and more. Firefly-R3572A provides one Type-C (USB2.0 OTG) port for ADB and also supports network ADB.

## Prepare the Connection [step]

Connect the PC to the board OTG port with a Type-C data cable; the OTG port location is shown in the [Interface Description](../product_introduction/started.md).

On Android, enable "Settings" → "Connected devices" → "Connect to PC". Then confirm the device is detected on the PC:

```bash
adb devices
```

The device status is expected to be `device`. If it shows `unauthorized`, confirm the debugging authorization dialog on the board.

## Network ADB [step]

When the board and the PC are on the same LAN, ADB can be used over the network without a cable:

1. Enable "Network ADB debugging" in "Developer options".
2. Check the board IP address, for example in "Settings" → "About device" → "Status".
3. Run on the PC:

```bash
adb connect <board-ip>:5555
adb shell
```

## Common Commands [step]

| Command | Purpose |
| --- | --- |
| `adb devices` | List connected devices |
| `adb shell` | Open a shell on the board |
| `adb logcat` | View system logs |
| `adb push <local> <remote>` | Upload a file to the board |
| `adb pull <remote> <local>` | Download a file from the board |
| `adb install <apk>` | Install an application |
| `adb reboot` | Reboot the board |
| `adb reboot loader` | Reboot into Loader upgrade mode |

### Connection Management

```bash
adb devices                       # list connected devices
adb tcpip 5555                    # restart adbd on the device listening on TCP port 5555
                                  # the USB cable can be unplugged afterwards
adb connect 192.168.1.100:5555    # connect to the device remotely, IP 192.168.1.100
adb disconnect                    # disconnect
```

## Debugging [step]

### Get System Logs: adb logcat

```bash
adb logcat                        # view all logs
adb logcat -s tag                 # view logs of a specific tag only
```

### Run Commands: adb shell

```bash
adb shell                         # enter the device command line
adb shell <command>               # execute a single command directly
```

#### Get Detailed Runtime Information: adb bugreport

```bash
adb bugreport                     # generate a detailed runtime report
adb bugreport report.zip          # save it locally for easy viewing with an editor
```

### root Permission

```bash
adb root                          # restart adbd with root permission (eng/userdebug firmware)
adb remount                       # remount the system partition as writable
```

## Application Management [step]

### Install an Application: adb install

```bash
adb install facebook.apk          # install facebook.apk
adb install -r twitter.apk        # upgrade twitter.apk
```

### Uninstall an Application: adb uninstall

```bash
adb uninstall com.example.app     # uninstall an application (by package name)
adb uninstall -k com.example.app  # uninstall but keep the data and cache directories
```

### Command Line Help: adb help

```bash
adb help                          # show the command line help of adb
```

## Install ADB on Windows [step]

1. Install the RK USB driver as described in [USB Firmware Upgrade](../getting_started/firmware_upgrade/upgrade_firmware.md).
2. Download and extract the Android SDK Platform-Tools, for example to `C:\platform-tools`.
3. Open a command prompt, switch to that directory and run:

```text
cd C:\platform-tools
adb devices
```

## Install ADB on Ubuntu [step]

```bash
sudo apt-get install android-tools-adb android-tools-fastboot
adb devices
```