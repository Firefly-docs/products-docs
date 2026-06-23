# 蜂窝网络

GS1-N2 支持 4G LTE, 在系统设置处，有多种网络形式，可以在此打开数据流量开关：

![](../../../gs1-n2_img/GS1-N2/4G.png)

在命令行生成网卡：

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