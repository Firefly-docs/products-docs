# 网络 IP 配置

## Debian 系统
在 Debian 9 系统，双网口中的 `eth0` 默认是动态获取 IP 的（即 DHCP），`eth1` 是固定成了 `192.168.150.1`。

`eth1` 的配置是通过 `/etc/network/interfaces.d/eth1` 文件完成的，这个文件是对 Debian 原始网络配置唯一的修改。

```shell
linaro@bm1684:~$ cat /etc/network/interfaces.d/eth1
auto eth1
iface eth1 inet static
      address 192.168.150.1
      netmask 255.255.255.0
      dns-nameservers 192.168.150.1
```

如果用户删掉这个文件，那么 `eth1` 会变成跟 `eth0` 一样的 DHCP 方式获取 IP；如果用户想要把 `eth0` 也固定 IP，就只需在 `/etc/network/interafces.d` 文件夹下创建一个 `eth0` 文件，重启后生效。

注意的是最好不要把两个网卡配置成同一网段，否则可能会出现问题。
## Ubuntu 系统
在 Ubuntu20.0 系统，双网口中的 `eth0` 默认是动态获取 IP 的（即 DHCP），`eth1` 是固定成了 `192.168.150.1`。

`eth1` 的配置是通过 `/etc/netplan/01-netcfg.yaml` 文件完成的，这个文件可对 Ubuntu 系统原始网络配置进行修改。

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

如果用户删掉这个文件，那么 `eth1` 会变成跟 `eth0` 一样的 DHCP 方式获取 IP；如果用户想要把 `eth0` 也固定 IP，就只需在 `/etc/netplan/01-netcfg.yaml` 文件中添加 `eth0` 节点（参考 `eth1`），重启生效即可。

