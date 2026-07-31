# Network IP configuration

## Debian System Configuration
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
## Ubuntu System Configuration
In Ubuntu 20.04, the `eth0` interface is set to obtain an IP address dynamically (DHCP), while `eth1` is configured with a fixed IP address of `192.168.150.1`.

The configuration for `eth1` can be found in the `/etc/netplan/01-netcfg.yaml` file. You can modify this file to adjust the network settings for your Ubuntu system.

```shell
linaro@bm1684:~$ cat /etc/netplan/01-netcfg.yaml  | more
network:
        version: 2
        renderer: networkd
        ethernets:
                eth1:
                        dhcp4: no
                        addresses: [192.168.150.1/24]
                        optional: yes
```
If the user deletes this file, then eth1 will revert to acquiring an IP address through DHCP, similar to eth0. If the user wants to assign a fixed IP address to eth0 as well, they only need to add an eth0 node (similar to eth1) in the /etc/netplan/01-netcfg.yaml file and then restart for the changes to take effect.

