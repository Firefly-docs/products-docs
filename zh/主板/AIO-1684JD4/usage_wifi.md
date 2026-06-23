# WIFI

AIO-1684JD4 支持无线 WIFI，在系统中网卡名默认为 `wlan0`：

```shell
root@bm1684:~# ifconfig wlan0
wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 20:57:9e:ba:02:fc  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## WIFI 连接

（1）使能 WIFI：

```shell
nmcli radio wifi on
```

（2）查看 WIFI 是否使能成功：

```shell
# 打印出 enabled 表示成功
nmcli radio wifi
```

（3）查看 WIFI 接入点：

```shell
nmcli dev wifi list
```

（4）连接到 WIFI 接入点：

```shell
sudo nmcli device wifi connect zouxftest1 password 12345678 name test
```
其中 `zouxftest1` 为 WIFI 接入点的名称，`12345678` 则是密码。

连接成功日志如下：

```shell
Device 'wlan0' successfully activated with 'fd6c634b-f517-4ae9-a8e9-292a9c19d25c'.
```

请注意，如果要禁用 WIFI 状态：

```shell
nmcli radio wifi off
```

## WIFI 热点

使用 `nmcli` 命令可以创建无线 AP 热点：

```shell
sudo nmcli device wifi hotspot ifname wlan0 con-name my-hostapt ssid zouxftest7 band bg password 12345678 channel 5
```

说明如下：
- `con-name`：连接名称，这里定义为 `my-hostapt`
- `ssid`：创建的 AP 热点的名称，这里定义为 `zouxftest7`
- `band`：WIFI 的协议标准，这里选择 `bg`
- `password`：创建的 AP 热点的密码，这里定义为 `12345678`
- `channel`：创建的 AP 热点的通过，这里定义为 `5`

在创建了无线 AP 热点以后，如果要打开/关闭 WIFI 热点：

```shell
sudo nmcli connection up[down] my-hostapt
```