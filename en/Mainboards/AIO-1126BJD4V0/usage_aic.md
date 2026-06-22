## Android in Container

AIC (Android in Container) means runing Android inside a container on Linux. RV1126B Linux can use docker to run Android and support running multiple Androids at the same time.

Cluster-Server with AIC can increase the amount of Androids, really helpful in cloud-mobile and cloud-gaming.

Here are some firmwares for testing: [Download](https://drive.google.com/drive/folders/1JvVj2VTJFpsZORyuEbG8oFtXP1ELOOJq?usp=sharing)

### Usage

Notice:

1. The host OS is Ubuntu Minimal without desktop, so using debug serial, ssh, adb shell etc. to interact.
2. Interaction with Android is through network adb and screen mirroring application like scrcpy, not by mouse/keyboard.
3. Operation AIC requires the following knowledge: basic Linux commands, docker commands, adb commands.

#### Create Containers

Firmwares come with an docker image, only need to create containers. Use script under /root/docker_sh/ to create:
```bash
./docker_sh/run_android.sh rk3588:firefly <id> <ipv4_address>

# example
./docker_sh/run_android.sh rk3588:firefly 0 192.168.100.1
```
`id` is a number you give to containers for easy management, id will be a part of the container name.

`ipv4_address` is the ip you give to containers, **it needs to be on the same subnet as the host and not in conflict with other devices or containers**.

If you need more Androids, just run it again, and remember to change the id and ip.

#### Connect to Containers

In the same local network, use any PC with adb to connect.
```bash
# <ip> is the target container's ip
adb connect <ip>

# After connection
# Start scrcpy
scrcpy -s <ip>
```
We skipped the tutorial of installing adb and scrcpy, please google it if you need.

#### Manage Containers

Use common docker commands to manage containers.
```bash
# Check all containers
root@firefly:~# docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED      STATUS                    PORTS     NAMES
cad5a331dea9   rk3588:firefly   "/init androidboot.h…"   6 days ago   Exited (137) 6 days ago             android_1
37f60c3b6b80   rk3588:firefly   "/init androidboot.h…"   6 days ago   Up 13 seconds                       android_0

# Start/Stop containers
docker start/stop <NAMES>

# Connect to Android shell(run exit to return)
docker exec -it <NAMES> sh

# Delete containers
docker rm <NAMES>
```
If the subnet of host changed, then you need to re-config the macvlan and change containers' ip.

Modify the parameters according to the actual situation.
```bash
docker network rm macvlan
docker network create -d macvlan --subnet=<SUBNET> --gateway=<GATEWAY> -o macvlan_mode=bridge -o parent=<PARENT> macvlan
```
Google for tutorial of changing docker container ip if you don't know.

### Performance

Run this cmd as root to enable performance mode to get better experience:
```bash
# It is normal to get an "Invalid argument", ignore it
root@firefly:~# echo performance | tee $(find /sys/devices -name *governor)
performance
tee: /sys/devices/system/cpu/cpuidle/current_governor: Invalid argument
```

Enter Android terminal or use adb shell to run this cmd can print the game fps:
```bash
# Notice: This cmd prints fps only when the game is running
setprop debug.sf.fps 1;logcat -s SurfaceFlinger
```

One RV1126B using AIC running two Genshin Impact with highest graphic setting at the same time can reach 35+ fps:
![](../../../rv1126b_img/common/aic_ys_performance.png)