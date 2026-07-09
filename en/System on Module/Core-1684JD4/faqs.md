
# FAQs


## How to Develop Qt Programs and Display Them via HDMI

The overall process is divided into two steps:  

1. Cross-compile Qt programs
2. Set up the Qt runtime environment and run the program

### Cross-compile

Taking a cross-compilation host with the system as `Ubuntu18.04` as an example, the steps are as follows:

#### Obtain Resources

Clone this [git repository](https://github.com/sophon-ai-algo/sophon-qt) using git clone.

Create and name a file called `se-build-server.yml` with the following content:

```
version: "3.3"
services:
    se-build-server:
        image: huangzhenming/se-build-server:V1.0
        container_name: sophon_build 
        privileged: true 
        network_mode: host 
        restart: always 
        volumes:
            - /data/your_working_dir:/workspace 
            - /data/toolchains/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu:/toolchains 
            - /etc/timezone:/etc/timezone:ro 
            - /etc/localtime:/etc/localtime:ro 
        command: ["/bin/ping", "localhost"] 
        user: linaro 
        working_dir: /workspace 
        environment: 
        - "PATH=$PATH:/toolchains/bin"
```    

 Download the necessary cross-compilation tools: [gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu](https://drive.google.com/drive/u/1/folders/1cC6hf_yKY8LsHSy1I8xbsR80fdVPSBaW)

Installation of the `docker` and `docker compose` tools varies from platform to platform, so please search for the relevant publicly available information on the Internet to install them, and we will not describe them here

Run docker (the runtime will automatically download the image from docker Hub, and the time required depends on the network speed, ensure a stable internet connection):
            
```
docker-compose -f se-build-server.yml  up -d
```  

After running, it will automatically create `/data/your_working_dir` and `/data/toolchains/` folders on your system, you can also modify the `yml` file to create folders in other paths.


Copy and extract the `git` package and the cross-compilation toolchain to the specified path:    
```
cp -r path/to/sophon-qt/qt-lib/qtbase-5.14.tgz /data/your_working_dir
cp -r path/to/sophon-qt/qt-example/SophUI/* /data/your_working_dir
cd /data/your_working_dir/ && tar xvf qtbase-5.14.tgz
cp -r path/to/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z /data/toolchains/
cd /data/toolchains/ && 7z x gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z
```  

* Cross-compile (using the `SophUI` source code as an example):    


    * enter docker：     


    ```
    docker ps -a  # Get the docker image and container information. The image name should have `huangzhenming` in it.
    docker exec -it  container_id  /bin/bash # enter docker 
    ```  


    * Perform the compilation(In docker):


    ```
    cd  /workspace
    ./install/bin/qmake SophUI.pro -spec linux-aarch64-gnu-g++ CONFIG+=qtquickcompiler
    make
    ```  


After compilation, the `SophUI` executable is generated, which can be executed directly on the device.  
            

### Run the Qt program

By following the above steps, you can get the executable file `SophUI` , which can be copied to the 1684 device by means of `scp`.

Before running, you need to write a script to set the environment variables as follows:  

```
#!/bin/sh
export QTDIR=/usr/lib/aarch64-linux-gnu 
export QT_QPA_FONTDIR=$QTDIR/fonts 
export QT_QPA_PLATFORM_PLUGIN_PATH=$QTDIR/qt5/plugins/
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
export QT_QPA_PLATFORM=linuxfb:fb=/dev/fl2000-0 
export QWS_MOUSE_PROTO=/dev/input/event3
./SophUI 
```  
After that, give the script permission to run, and execute it with the HDMI monitor plugged in.

**Note that if the 1684 is running on a `Debian` system, you will need to import the Qt libraries and other files separately, otherwise you will get an error, which is not the case with Ubuntu systems**.

Import it from the PC via `scp` and execute it on the 1684 device:  

```
scp PC@IP:/path/to/qtbase-5.14/lib/* /usr/lib/aarch64-linux-gnu/ 
mkdir -p /usr/lib/aarch64-linux-gnu/qt5
scp  PC@IP:/path/to/qtbase-5.14/plugins/ /usr/lib/aarch64-linux-gnu/qt5/
```
If the Qt program also contains fonts, you also need to [download](https://drive.google.com/drive/u/1/folders/1cC6hf_yKY8LsHSy1I8xbsR80fdVPSBaW) the font files here, and then copy them to the `/usr/lib/aarch64-linux-gnu/font` directory on the 1684 device. For example: `/usr/lib/aarch64-linux-gnu/fonts/wqy-microhei.ttc`.


## IP configuration of the ethernet interface

- The shell is marked with "1", which is close to the network port of the RS232/RS485 interface, corresponding to the system `eth0` network interface, set to DHCP mode.
- The shell is marked with "2", which is close to the Ethernet port of the HDMI interface, corresponding to the system `eth1` network interface, set to static address mode: `192.168.150.1/24`.
## What is the default username and password of the system?

* Username: `linaro`
* Password: `linaro`
* Switch superuser: `sudo -s`

## What should I do if the startup is abnormal and the cycle restarts?

It may be that the power supply current is not enough, please use a power supply with a voltage of 12V and a current of more than 5A.

