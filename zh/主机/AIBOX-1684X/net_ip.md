# 网络 IP 配置


在默认情况中，AIBOX-1684X 的网口 0 设置了动态 IP，网口 1 设置了静态 IP。这些配置都是通过系统中的 `/etc/netplan/01-netcfg.yaml` 文件实现的。

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

用户可通过修改 `/etc/netplan/01-netcfg.yaml` 文件达到修改网络配置的目的，举例如：

- 配置网口 0 为静态 IP：参考原 `eth1` 设置修改 `eth0` 节点，将 `dhcp4` 参数修改为 `no`，并添加 IP 地址进 `addresses` 参数中。

- 配置网口 1 为动态 IP：参考原 `eth0` 设置修改 `eth1` 节点，将 `dhcp4` 参数修改为 `yes`，并去除 `addresses` 参数中的 IP 地址。

在完成 `/etc/netplan/01-netcfg.yaml` 文件的修改后，用户可以通过执行 `sudo netplan apply` 命令使设置立即生效。

