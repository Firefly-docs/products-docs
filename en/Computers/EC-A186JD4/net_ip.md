# Network IP Configuration

## Ubuntu System
In the Ubuntu 20.04 system, the `eth0` interface in the dual network ports is configured to obtain an IP dynamically (i.e., via DHCP), while `eth1` is fixed to `192.168.150.1`.

Network configuration is done through the `/etc/netplan/01-netcfg.yaml` file, which allows modifications to the original network configuration of the Ubuntu system.

```shell
linaro@sophon:~$ cat /etc/netplan/01-netcfg.yaml
network:
        version: 2
        renderer: networkd
        ethernets:
                eth0:
                        dhcp4: yes
                        addresses: []
                        optional: yes
                        dhcp-identifier: mac
                eth1:
                        dhcp4: no
                        addresses: [192.168.150.1/24]
                        optional: yes
```

If the user deletes this file, then `eth1` will change to obtain an IP using the same DHCP method as `eth0`; if the user wants to set a fixed IP for `eth0` as well, they only need to modify the `eth0` node in the `/etc/netplan/01-netcfg.yaml` file (referencing `eth1`), and restart for the changes to take effect.