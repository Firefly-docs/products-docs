# Login

There are two ways to login to AIO-1684XQ, one is via RS232, the other is via network SSH.

## RS232

Through the RS232 login method, users can directly refer to "[Using RS232 serial port debugging](https://wiki.t-firefly.com/en/iCore-1684XQ/debug.html#shi-yong-rs232-chuan-kou-tiao-shi)", the login account and password are `linaro`.

## Ethernet SSH

AIO-1684XQ has two Ethernet interfaces. By default, network port 0 (near the serial port) is set with dynamic IP, while network port 1 (near the HDMI port) is set with static IP `192.168.150.1`, subnet mask `255.255 .255.0`, you can set the PC to `192.168.150.2/24` for initial login.

Network port 0 is assigned an IP by the router, and the IP address information is generally unknown in advance, so it is best for users to select network port 1 for the first login.

After the network port light flashes normally, open the terminal and use `ssh` to log in, the port number is `22`, and the username and password are also `linaro`:

   ```shell
   ssh linaro@192.168.150.1
   ```

If the login fails, you can try to use the PC to `ping` the IP address of the AIO-1684XQ network port 1. Note that the PC needs to add the IP address of the same network segment, because the IP address of the PC is not necessarily in the same network segment .

The following is how to add the same network segment IP on PC (using administrator mode):

- windows 10

   ```
   netsh int ipv4 add address "Ethernet" 192.168.150.101 255.255.255.0
   ```
   Where `"Ethernet"` refers to the network interface, `192.168.150.101` is the IP address, and `255.255.255.0` is the subnet mask.

- linux
      
   ```shell
   ifconfig enp4s0:1 192.168.150.89
   ```
   Among them, `enp4s0` is the name of the network card driver of the PC, and you need to execute the `ifconfig` command to view it.