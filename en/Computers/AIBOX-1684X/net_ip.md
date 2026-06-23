# Network IP configuration


By default, AIBOX-1684X has a dynamic IP set for ethernet port 0, and a static IP set for ethernet port 1. These configurations are implemented through the `/etc/netplan/01-netcfg.yaml` file in the system.

```shell
linaro@aibox-1684x:~$ cat /etc/netplan/01-netcfg.yaml
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

Users can modify the network configuration by modifying the `/etc/netplan/01-netcfg.yaml` file, for example:

- Set ethernet port 0 to a static IP address: Refer to the original setting of `eth1` to change node `eth0`, change parameter `dhcp4` to no, and add the IP address to parameter `addresses`.

- Set ethernet port 1 to a dynamic IP address: Refer to the original setting of `eth0` to change node `eth1`, change parameter `dhcp4` to yes, and remove the IP address in the parameter `addresses`.

After modifying the '/etc/netplan/01-netcfg.yaml' file, users can make the Settings take effect immediately by executing the `sudo netplan apply` command.

