# Network IP configuration

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

