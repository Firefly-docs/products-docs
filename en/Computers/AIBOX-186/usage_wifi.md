# WIFI

AIBOX-186 supports wireless WIFI, and the default name of the network card in the system is `wlanx` or `wlPxp1s0` (depending on different wifi modules). Taking `wlan0` as an example:

```shell
linaro@sophon:~# ifconfig wlan0
wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 20:57:9e:ba:02:fc  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### WIFI Connection

(1) Enable WIFI:

```shell
nmcli radio wifi on
```

(2) Check if WIFI is successfully enabled:

```shell
# Printing out enabled indicates success
nmcli radio wifi
```

(3) Check WIFI access points:

```shell
nmcli dev wifi
```

(4) Connect to a WIFI access point:

```shell
sudo nmcli device wifi connect zouxftest1 password 12345678 name test
```
Here, `zouxftest1` is the name of the WIFI access point, and `12345678` is the password.

The successful connection log is as follows:

```shell
Device 'wlan0' successfully activated with 'fd6c634b-f517-4ae9-a8e9-292a9c19d25c'.
```

Please note, if you want to disable WIFI:

```shell
nmcli radio wifi off
```

### WIFI Hotspot

You can create a wireless AP hotspot using the `nmcli` command:

```shell
sudo nmcli device wifi hotspot ifname wlan0 con-name my-hostapt ssid zouxftest7 band bg password 12345678 channel 5
```

Explanation as follows:
- `con-name`: Connection name, defined here as `my-hostapt`
- `ssid`: Name of the created AP hotspot, defined here as `zouxftest7`
- `band`: WIFI protocol standard, chosen here as `bg`
- `password`: Password for the created AP hotspot, defined here as `12345678`
- `channel`: Channel for the created AP hotspot, defined here as `5`

After creating the wireless AP hotspot, if you want to turn the WIFI hotspot on/off:

```shell
sudo nmcli connection up[down] my-hostapt
```