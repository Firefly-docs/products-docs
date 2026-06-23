# Get Started

## Power On

AIBOX-1688 will turn on automatically when connected to the power supply. If the power supply is connected, please press the power button to turn on the machine.

When the board boots into the Linux system, the LED will be turn on with green color.

## Login

There are two ways to log in to the AIBOX-1688 terminal:
1. through the Type-C serial port
2. through the Ethernet

### Type-C Serial Port

To login through the Type-C serial port, prepare a Type C to USB cable and connect it to the Type-C serial port of AIBOX-1688 and the USB port of the PC.

Users can use software such as [MobaXterm](https://mobaxterm.mobatek.net/download-home-edition.html) or minicom to connect to the serial port.

The serial port parameters are:
- Baud Rate: 115200 bps
- Data Bits: 8
- Stop Bits: 1
- Parity: none
- Flow-Control: none

The user name and password used to log in to the terminal are both `linaro`.

### Network Remote Login

Before using the network remote login, check the IP address of AIBOX-1688 ethernet port:

- Ethernet port 0 (near the 12V power port) has a dynamic IP address. Log in to the terminal using the Type-C serial port and run the `ifconfig` command to check the IP address of ethernet port 0 (eth0).
- Ethernet port 1 (near the USB port) is set to the static IP `192.168.150.1` with subnet mask `255.255.255.0`. You can set your PC to `192.168.150.2/24` for initial access.

You can easily change the IP address of a PC in the following ways:

- Windows (Open cmd as an administrator and run)：

```bash
netsh int ipv4 set interface "Ethernet" dhcpstaticipcoexistence=enabled
netsh int ipv4 add address "Ethernet" 192.168.150.2 255.255.255.0
```

- Linux

```bash
sudo ifconfig enp4s0:1 192.168.150.2   # replace enp4s0 with the actual ethernet network interface
```

After confirming the IP address of the ethernet port, if you can successfully `ping` the IP address of the ethernet port on the PC side, then you can use `ssh` to log in:

```
# 192.168.150.1 must be replaced with the actual IP address of the ethernet port
ssh linaro@192.168.150.1
```

The user name and password are also `linaro`:

To change the IP address of the network port, see [Network IP configuration](net_ip.md)

## Power Off

Note: Please complete the software/hardware shutdown before disconnecting the power, so as not to damage the filesystem data.

* Software shutdown: Run `sudo poweroff` in the terminal.

* Hardware shutdown: Press and hold the power button until the LED stops flashing.

When the fan stops running and the indicator LED is off, AIBOX-1688 is powered off. Then you can safely disconnect the power supply.