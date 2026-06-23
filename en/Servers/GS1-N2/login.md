# Login

|Module|User|Password|
|----|----|----|
|RK3588(Core-3588JD4)<br>RK3576(Core-3576JD4)|firefly|firefly|
|NVIDIA(Jetson Orin Nano)<br>NVIDIA(Jetson Orin NX)|nvidia|nvidia|
|SOPHGO(Core-1688JD4)|linaro|linaro|

## Main Module RK3588(Core-3588JD4)
### HDMI or VGA 
Connect the monitor.
### login to sub from main
Main module use console(node `/dev/ttyS1`, baudrate `115200`) to login sub module.
```
sudo minicom -b 115200 -D /dev/ttyS1
```

## Main Module RK3576(Core-3576JD4)
### HDMI or VGA 
Connect the monitor.
### login to sub from main
Main module use console(node `/dev/ttyS8`, baudrate `115200`) to login sub module.
```
sudo minicom -b 115200 -D /dev/ttyS8
```

## Main Module NVIDIA(Jetson Orin Nano/Jetson Orin NX)
### HDMI or VGA 
Connect the monitor.
### login to sub from main
Main module use console(node `/dev/ttyTHS1`, baudrate `115200`) to login sub module.
```
sudo minicom -b 115200 -D /dev/ttyTHS1
```