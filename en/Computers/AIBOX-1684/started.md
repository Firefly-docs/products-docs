# First use

## Power On

AIBOX-1684 will turn on automatically when connected to the power supply. If the power supply is connected, please press the power button to turn on the machine.

## Login Terminal

There are two ways to log in to the AIBOX-1684 terminal. One is to log in through the Type C serial port, and the other is to log in remotely through the network.

### Type C Serial Port Login

To log in through the Type C serial port, prepare a Type C to USB cable and connect it to the Type C port of AIBOX-1684 and the USB port of the PC.

Users can use software such as [MobaXterm](https://mobaxterm.mobatek.net/download-home-edition.html) or minicom to connect to the serial port. The baud rate of the serial port is 115200. The user name and password used to log in to the terminal are both `linaro`.

### Network Remote Login

Before using the network remote login, check the IP address of AIBOX-1684 ethernet port:

Ethernet port 0 (near the USB port) has a dynamic IP address. Log in to the terminal using the Type C serial port and run the `ifconfig` command to check the IP address of ethernet port 0 (eth0).

Ethernet port 1 (near the 12V power port) is set to the static IP `192.168.150.1`, subnet mask `255.255.255.0`. you can set the PC to `192.168.150.2/24` for initial access.

You can easily change the IP address of a PC in the following ways:

- Windows (Open cmd as an administrator and run)：

```bash
netsh int ipv4 set interface "Ethernet" dhcpstaticipcoexistence=enabled
netsh int ipv4 add address "Ethernet" 192.168.150.2 255.255.255.0
```

- Linux

```bash
ifconfig enp4s0:1 192.168.150.2
```

After confirming the IP address of the ethernet port, if you can successfully `ping` the IP address of the ethernet port on the PC side, then you can use `ssh` to log in, where the port number is 22, and the user name and password are also `linaro`:

```
# 192.168.150.1 must be replaced with the actual IP address of the ethernet port
ssh linaro@192.168.150.1
```

To change the IP address of the network port, see [Network IP configuration](net_ip.md)

## Power Off

Note: Please complete the software/hardware shutdown before disconnecting the power, so as not to damage the file system data.

* Software shutdown: Run `sudo poweroff` in the terminal.

* Hardware shutdown: Press and hold the power button until the indicator light stops flashing.

When the fan stops running and the indicator light is off, AIBOX-1684 is powered off. You can safely disconnect the power supply.