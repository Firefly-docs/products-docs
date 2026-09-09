# 网络 IP 配置

## Ubuntu 系统
在 Ubuntu20.0 系统，双网口中的 `eth0` 默认是动态获取 IP 的（即 DHCP），`eth1` 是固定成了 `192.168.150.1`。

网络配置是通过 `/etc/netplan/01-netcfg.yaml` 文件完成的，这个文件可对 Ubuntu 系统原始网络配置进行修改。

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

如果用户删掉这个文件，那么 `eth1` 会变成跟 `eth0` 一样的 DHCP 方式获取 IP；如果用户想要把 `eth0` 也固定 IP，就只需在 `/etc/netplan/01-netcfg.yaml` 文件中修改 `eth0` 节点（参考 `eth1`），重启生效即可。

