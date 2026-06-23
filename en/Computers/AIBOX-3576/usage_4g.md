# Cellular Network

AIBOX-3576 supports 4G LTE. In system settings, there are multiple network options. You can enable mobile data here:

![](../../../aibox_img/AIBOX-3576/4G.png)

Check the network interface via command line:

```shell
$ ifconfig wwan0
wwan0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.176.252.100  netmask 255.255.255.248  destination 10.176.252.100
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
        RX packets 4  bytes 405 (405.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 439 (439.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```