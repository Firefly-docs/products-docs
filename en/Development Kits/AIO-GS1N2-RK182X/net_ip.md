# Network IP configuration

In Debian 9, `eth0` in the dual network port defaults to obtain IP dynamically (that is, DHCP), and `eth1` is fixed to `192.168.150.1`.

The configuration of `eth1` is done via the `/etc/network/interfaces.d/eth1` file, which is the only modification to the original Debian network configuration.

```shell
linaro@bm1684:~$ cat /etc/network/interfaces.d/eth1
auto eth1
iface eth1 inet static
       address 192.168.150.1
       netmask 255.255.255.0
       dns-nameservers 192.168.150.1
```

If the user deletes this file, then `eth1` will become the same as `eth0` to obtain the IP through DHCP; If the user wants to fix the IP of `eth0`, just create an `eth0` file in the `/etc/network/interafces.d` folder, and it will take effect after restarting.

Note that it is best not to configure the two network cards as the same network segment, otherwise problems may occur.

