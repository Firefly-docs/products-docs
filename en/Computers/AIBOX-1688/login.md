# Login
There are two ways to log in to AIBOX-1688: one is through Type-C, and the other is via network SSH.

## Type-C

For the Type-C login method, users can directly refer to the AIO-1688JD4 wiki page's section on "[Using Type-C Serial Debugging](https://wiki.t-firefly.com/en/Core-1688JD4/debug.html#using-type-c-for-serial-port-debugging)," with the login username and password both being `linaro`.

## Ethernet SSH

AIBOX-1688 has two Ethernet ports. By default, at the factory, port 0 (near the USB connector) is set to dynamic IP, while port 1 (near the HDMI port) is set to static IP `192.168.150.1` with a subnet mask of `255.255.255.0`. You can set the PC to `192.168.150.2/24` for the initial login.

Port 0 has its IP assigned by a router, and the IP address information is generally not known beforehand, so it is best for the user to choose port 1 for the initial login.

After the port light is flashing normally, open a terminal and use `ssh` to log in, with the port number being `22`, and the username and password also being `linaro`:

```shell
ssh linaro@192.168.150.1
```

If the login fails, you can try to see if the PC can `ping` the IP address of AIBOX-1688's port 1. Note that the PC must have an IP in the same subnet, as the PC's IP address may not necessarily be in the same subnet.

Here are the methods for adding a same-subnet IP on the PC (using administrator mode):

- Windows 10

  ```
  netsh int ipv4 add address "Ethernet" 192.168.150.101 255.255.255.0
  ```
  Here, `"Ethernet"` refers to the network interface, `192.168.150.101` is the IP address, and `255.255.255.0` is the subnet mask.

- Linux

  ```shell
  ifconfig enp4s0:1 192.168.150.89
  ```
  Here, `enp4s0` is the network card driver name of the PC, which can be checked by executing the `ifconfig` command.

