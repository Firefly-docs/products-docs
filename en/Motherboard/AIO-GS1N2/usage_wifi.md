# WIFI

GS1-N2 supports wireless WIFI, and the default name of the network card in the system is `wlanx` or `wlPxp1s0` (depending on different wifi modules). Taking wlan0 as an example:

```shell
# ifconfig wlan0
wlan0: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500
         ether 20:57:9e:ba:02:fc txqueuelen 1000 (Ethernet)
         RX packets 0 bytes 0 (0.0 B)
         RX errors 0 dropped 0 overruns 0 frame 0
         TX packets 0 bytes 0 (0.0 B)
         TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

## WIFI connection

(1) Enable WIFI:

```shell
nmcli radio wifi on
```

(2) Check whether WIFI is enabled successfully:

```shell
# Print out enabled to indicate success
nmcli radio wifi
```

(3) View WIFI access point:

```shell
nmcli dev wifi list
```

(4) Connect to WIFI access point:

```shell
sudo nmcli device wifi connect zouxftest1 password 12345678 name test
```
Where `zouxftest1` is the name of the WIFI access point, and `12345678` is the password.

The connection success log is as follows:

```shell
Device 'wlan0' successfully activated with 'fd6c634b-f517-4ae9-a8e9-292a9c19d25c'.
```

Note that if you want to disable WIFI status:

```shell
nmcli radio wifi off
```

## WIFI hotspot

Use the `nmcli` command to create a wireless AP hotspot:

```shell
sudo nmcli device wifi hotspot ifname wlan0 con-name my-hostapt ssid zouxftest7 band bg password 12345678 channel 5
```

described as follows:
- `con-name`: The name of the connection, defined here as `my-hostapt`
- `ssid`: the name of the created AP hotspot, defined here as `zouxftest7`
- `band`: WIFI protocol standard, choose `bg` here
- `password`: the password of the created AP hotspot, defined here as `12345678`
- `channel`: the pass through of the created AP hotspot, defined as `5` here

After creating a wireless AP hotspot, if you want to turn on/off the WIFI hotspot:

```shell
sudo nmcli connection up [down] my-hostapt
```