# Technical cases

## OpenSIP build

SIP (Session Initiation Protocol) was developed to help provide advanced telephone services across the Internet. Internet telephony (IP telephony) is evolving to a formal business telephony model, and SIP is an important member of the NGN (Next Generation Network) series of protocols required to ensure the realization of this evolution. Support H.264 protocol.

*  The system used this time is Ubuntu18.04 (if you use ubutu20.04 or Debian, etc., you may need to modify it slightly)

### Compile opensips

Install the required environment pack
```
sudo apt-get install flex bison libncurses5-dev

#Install MySQL 5.7 version database
sudo apt-get install mysql-server
sudo apt-get install mysql-client
sudo apt-get install libmysqlclient-dev

git clone https://github.com/OpenSIPS/opensips.git -b2.2 opensips-2.2
```


Configuration before compilation:

* Add db_mysql module function and save
* Modify the installation directory PREFIX=/usr/local/opensips/
```
# unset C_INCLUDE_PATH #If the compiling object menuconfig is unsuccessful, try to execute this command
make menuconfig

---------------------------------------------
|                                           |
|    --->  Configure Compile Options        |
|          Compile And Install OpenSIPS     |
|          Cleanup OpenSIPS sources         |
|          Generate OpenSIPS Script         |
|          Exit & Save All Changes          |
---------------------------------------------

Choose
Configure Compile Options
    Configure Excluded Modules
        [*] db_mysql #Check MySQL support module
    Configure Install Prefix
        /usr/local/opensips/    #It is recommended to modify the installation path, it is easy to uninstall opensips in the future
```

Compile:
* make all -j6
* make install

### sips service configuration
#### 2.1 Modify opensips.cfg
```
---------------------------------------------
|                                           |
|          Configure Compile Options        |
|          Compile And Install OpenSIPS     |
|          Cleanup OpenSIPS sources         |
|    --->  Generate OpenSIPS Script         |
|          Exit & Save All Changes          |
---------------------------------------------

Generate OpenSIPS Script
    Residential Script
        Configure Residential Script    #1. Configure script first
            #Add to:
            [*] ENABLE_TCP
            [*] USE_AUTH
            [*] USE_DBACC
            [*] USE_DBUSRLOC
            [*] USE_DIALOG
        Save Residential Script         #2. Save the configuration
        Generate Residential Script     #3. Generate script
```

* Replace opensips.cfg: cp opensips-2.2/etc/opensips_residential_2021-5-6_7:21:6.cfg opensips.cfg
* Modify the mpath variable in opensips.cfg: mpath="/usr/local/opensips/lib64/opensips/modules/"
* Modify listen to the machine IP: listen=udp:127.0.0.1:5060 # CUSTOMIZE ME listen=tcp:127.0.0.1:5060 # CUSTOMIZE ME

**Check whether the configuration file syntax is passed**

```
sbin/opensips -C

May  6 07:30:20 [6441] INFO:core:fix_poll_method: using epoll_lt as the IO watch method (auto detected)
Listening on
             udp: 168.168.102.131 [168.168.102.131]:5060
             tcp: 168.168.102.131 [168.168.102.131]:5060
Aliases:

May  6 07:30:20 [6441] NOTICE:core:main: config file ok, exiting...
```

#### 2.2 Modify opensipsctlrc
```
SIP_DOMAIN=168.168.102.131
DBENGINE=MYSQL
#localhost
DBHOST=168.168.102.131
DBNAME=opensips
DBRWUSER=opensips
DBRWPW="opensipsrw"
DBROOTUSER="root"
```

#### 2.3 Create opensips database
```
root@firefly:/usr/local/opensips/sbin# ./opensipsdbctl create
MySQL password for root:
INFO: test server charset
INFO: creating database opensips ...
INFO: Using table engine InnoDB.
INFO: Core OpenSIPS tables successfully created.
Install presence related tables? (y/n): y
INFO: creating presence tables into opensips ...
INFO: Presence tables successfully created.
Install tables for imc cpl siptrace domainpolicy carrierroute userblacklist b2b cachedb_sql registrant call_center fraud_detection emergency? (y/n): y
INFO: creating extra tables into opensips ...
INFO: Extra tables successfully created.
```

#### 2.4 Add sip user

* ./opensipsctl add username password       #Add user
```
./opensipsctl add 1000 1000     #Add an opensips user whose account and password are both ‘1000’
./opensipsctl add 2000 2000     #Add an opensips user whose account and password are both ‘2000’
./opensipsctl add 3000 3000     #Add an opensips user whose account and password are both ‘3000’
```

* View the addition of the opensips database in MySQL
```
mysql -u opensips

show databases;
use opensips;
select * from subscriber;
```

Specific operation process:
```

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| opensips           |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use opensips;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> use opensips;
Database changed
mysql> select * from subscriber;

| id | username | domain          | password | email_address | ha1                              | ha1b                             | rpid |

|  1 | 1000     | 168.168.102.131 | 1000     |               | 723edede4ecefd0b5762a9d17e32901a | e6c216c8db0d13ce066491f19164a12b | NULL |
|  2 | 2000     | 168.168.102.131 | 2000     |               | 6d3515702693940f19e95e82be498e42 | 10299abcbca6bee63078df877918e557 | NULL |
|  3 | 3000     | 168.168.102.131 | 3000     |               | 1d3a4a80dc09cecea466877fea882811 | 8255c1a861f98b431dc621166130c9d8 | NULL |

3 rows in set (0.01 sec)
```

### 3. Start the sips service
* Start the opensips service: sudo service mysql restart

### 4. Test the remote communication function

Use [linphone](https://new.linphone.org/technical-corner/linphone?qt-technical_corner=2#qt-technical_corner)

* Enter the opensips account address
Account number: 1000
Password: 1000

![](../../../rk3399_img/opensips1.png)
![](../../../rk3399_img/opensips2.png)
![](../../../rk3399_img/opensips3.png)

* Test Results:
**It can carry out video, voice calls, text message transmission and file transmission.**

### FAQ
* Can't connect to MySQL server on '168.168.100.178' (111) root@jinchengubuntu-VirtualBox:/usr/local/opensips/sbin# ./opensipsdbctl create MySQL password for root: INFO: test server charset mysql: [Warning] Using a password on the command line interface can be insecure. ERROR 2003 (HY000): Can't connect to MySQL server on '168.168.100.178' (111) mysql: [Warning] Using a password on the command line interface can be insecure. ERROR 2003 (HY000): Can't connect to MySQL server on '168.168.100.178' (111) WARNING: Failed to get the available and used character sets

Comment code
```
root@jinchengubuntu-VirtualBox:/etc/mysql# grep "bind-address" * -nR
mysql.conf.d/mysqld.cnf:43:bind-address         = 127.0.0.1

sudo service mysql restart
```
## MIPI camera

The RK3399 board with `MIPI CSI` interface has added the support of dual MIPI camera `OV13850`, and the camera example is also added in the application. The following describes the relevant configuration.

### Device tree file configuration

Take `rk3399-firefly-aiojd4.dts` as an example, these are the configurations that the camera needs to open. Since the kernel has added support for dual MIPI cameras, there are two cameras configured here. If you only use one camera, you only need to open one of the cameras.

```
//Power Management
&vcc_mipi {
        status = "okay";
};

&dvdd_1v2 {
        status = "okay";
};

//MIPI camera 1
&ov13850 {
        status = "okay";
};
&rkisp1_0 {
        status = "okay";
};

&mipi_dphy_rx0 {
        status = "okay";
};

&isp0_mmu {
        status = "okay";
};
//MIPI camera 2
&ov13850_1 {
        status = "okay";
};
&rkisp1_1 {
        status = "okay";
};
&mipi_dphy_tx1rx1 {
        status = "okay";
};

&isp1_mmu {
        status = "okay";
};
```

**Note:** If you are using the `core-3399` core board plus a DIY backplane, you may need to modify the corresponding GPIO attributes.

### Debug

Before running the script, confirm whether the `OV13850` device is successfully registered. The following is the successful kernel log:

```
root@firefly:~# dmesg | grep ov13850
//MIPI camera 1
[1.276762] ov13850 1-0036: GPIO lookup for consumer reset
[1.276771] ov13850 1-0036: using device tree for GPIO lookup
[1.276803] of_get_named_gpiod_flags: parsed'reset-gpios' property of node'/i2c@ff110000/ov13850@36[0]'-status (0)
[1.276855] ov13850 1-0036: Looking up avdd-supply from device tree
[1.277034] ov13850 1-0036: Looking up dovdd-supply from device tree
[1.277170] ov13850 1-0036: Looking up dvdd-supply from device tree
[1.277535] ov13850 1-0036: GPIO lookup for consumer pwdn
[1.277544] ov13850 1-0036: using device tree for GPIO lookup
[1.277575] of_get_named_gpiod_flags: parsed'pwdn-gpios' property of node'/i2c@ff110000/ov13850@36[0]'-status (0)
[1.281862] ov13850 1-0036: Detected OV00d850 sensor, REVISION 0xb2
//MIPI camera 2
[1.284442] ov13850 1-0046: GPIO lookup for consumer pwdn
[1.284461] ov13850 1-0046: using device tree for GPIO lookup
[1.284523] of_get_named_gpiod_flags: parsed'pwdn-gpios' property of node'/i2c@ff110000/ov13850@46[0]'-status (0)
[1.288235] ov13850 1-0046: Detected OV00d850 sensor, REVISION 0xb2
```

Related device files should be generated under `/dev`:

```
root@firefly:~# ls /dev/video
//MIPI camera 1
video0 video1 video2 video3
//MIPI camera 2
video4 video5 video6 video7

```
**Note:** Similarly, if you only use one camera, you only need to register one camera device.

### Test the preview camera

#### v4l2
Use the v4l2 method to preview the camera:
```
apt-get install -y git libopencv-dev cmake libdrm-dev g++ librga-dev

git clone https://github.com/T-Firefly/rkisp-v4l2.git

cd rkisp-v4l2/rkisp_demo/
cmake ./
make

sudo -u firefly DISPLAY=:0 ./rkisp_demo -c 300 -d /dev/video0 -w 640 -h 480
or
sudo -u firefly DISPLAY=:0 ./rkisp_demo -c 300 -d /dev/video5 -w 640 -h 480
```


#### gstreamer
Use the gstreamer method to preview the camera:
```
gst-launch-1.0 v4l2src device=/dev/video0 ! video/x-raw,format=NV12,width=640,height=480, framerate=30/1 ! videoconvert ! kmssink &
```

Just run the script, and the result is as shown in the figure:

![](../../../rk3399_img/mipi_csi.jpg)



#### FFMPEG read MIPI camera

* Multiplexing using v4l2loopback and FFMPEG
```
apt-get install -y git libopencv-dev cmake libdrm-dev g++ librga-dev python-dev

git clone https://github.com/umlaeute/v4l2loopback.git

cd v4l2loopback/

#Refer to "Installing linux-headers and linux-image"
make install
...

//Create video10, video11 and video12 devices using v4l2loopback
sudo insmod v4l2loopback.ko videonr=10,11,12

#Open and read the video0 device, and write the image information into video10
sudo -u firefly DISPLAY=:0  ./rkisp_demo -c 300  -d /dev/video0 -w 640  -h 480 -D /dev/video10

// Use the ffmpeg copy function to copy the content in video10 to video11 and video12 devices
ffmpeg -loglevel quiet -f v4l2 -video_size 640x480 -r 10 -i /dev/video10 -preset fast -codec copy -f v4l2 /dev/video11 -preset fast -codec copy -f v4l2 /dev/video12

// Start another terminal and use the ffplay tool to preview video11 or video12 at will
ffplay -i /dev/video11
```


#### OpenCV reading MIPI camera

The MIPI camera is **not supported** OpenCV **capture.open(index)** The way to read, here are the methods to read the camera in 2:

In this way, **it is recommended to use the C++ version of OpenCV**. For details, please refer to the previous v4l2 preview section. v4l2_simple_demo is a simplified version of rkisp_demo.cpp, please refer to rkisp_demo.cpp for details.

* C++

    **Recommended to use the C++ version of OpenCV**, please refer to the previous v4l2 preview section for details. v4l2_simple_demo is a simplified version of rkisp_demo.cpp, please refer to rkisp_demo.cpp for details.
    ```
    Refer：
    https://github.com/T-Firefly/rkisp-v4l2.git

    git clone https://github.com/T-Firefly/rkisp-v4l2.git
    cd rkisp-v4l2/mipi_video_demo/v4l2_simple_demo
    make
    # open /dev/video0
    sudo -u firefly DISPLAY=:0 ./opencv
    ```

* Python

    Using OpenCV-Python to read MIPI cameras needs to add Gstreamer support, and needs to reproduce the compilation and installation of OpenCV. Refer to ["OpenCV Compilation and Installation"](OpenCV_support.md)
    ```
    git clone https://gitlab.com/firefly-linux/test_code/rkisp-v4l2.git
    cd mipi_video_demo/OpenCV_Python
    # Please read rkisp-v4l2/mipi_video_demo/OpenCV_Python/README.md carefully.
    python3 opencv_gst_test.py
    ```

## OpenCV compilation and installation

Many customers encounter problems with OpenCV, and they mostly focus on how to obtain the data of the mipi camera. Because the V4l2 protocol used by OpenCV is different from the mipi camera driver protocol written by Rockchip, the camera API of OpenCV cannot be used directly. This section describes how to compile Opencv and add GStreamer API support.

### Require
* OS： Ubuntu18.04 / Debian 10
* OpenCV version： 3.4.15
* Board: RK3399

### Compile and install
1. Build a python3.7 virtual environment
    ```
    # Install compilation environments such as gcc and cmake in turn
    # Install python3.7-tk and python3.7-dev
    # Install virtualenv virtual environment
    sudo apt install gcc cmake git build-essential \
    python3-tk python3.7-dev \
    virtualenv
    ```

1. Create a python3.7 virtual environment
    ```
    virtualenv -p /usr/bin/python3.7m /home/firefly/venv
    # Use a virtual environment, if you want to quit, you can enter deactivate in the terminal
    source /home/firefly/venv/bin/activate
    ```

1. Install the required environment packages for Opencv
    ```
    # Install the compilation environment, gtk package and related codec libraries
    sudo apt install cmake build-essential libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

    # Notice! If rknn_toolkit(or lite) is installed, you do not need to install Python-numpy, otherwise rknn_toolkit(or lite) will be unavailable
    sudo apt install python-numpy
    ```

1. Download Opencv
    ```
    # Create folder
    mkdir opencv
    cd opencv

    # Download opencv-3.4.15.zip
    wget https://github.com/opencv/opencv/archive/refs/tags/3.4.15.zip
    unzip opencv-3.4.15.zip
    ```

1. Configure
    ```
    mkdir build
    cd build

    # Making connections between OpenCV and Python3
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D 	PYTHON_DEFAULT_EXECUTABLE=$(python -c "import sys; print(sys.executable)")\
    -D PYTHON3_EXECUTABLE=$(python -c "import sys; print(sys.executable)")  \
    -D PYTHON3_NUMPY_INCLUDE_DIRS=$(python -c "import numpy; print (numpy.get_include())")  \
    -D PYTHON3_PACKAGES_PATH=$(python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())")  \
    ../opencv-3.4.15
    ```

1. To support Gstreamer API, perform the following configuration operations, otherwise skip
    ```
    sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
    cmake -D WITH_GSTREAMER=ON ../opencv-3.4.15
    ```
1. Compile and install
    ```
    # If the board memory does not exceed 2G, it is recommended not to exceed -j4
    make -j6
    # The installation process requires sudo privileges
    sudo make install
    ```

### Test
1. Test
    ```
    source /home/firefly/venv/bin/activate
    git clone https://gitlab.com/firefly-linux/test_code/rkisp-v4l2.git
    cd mipi_video_demo/OpenCV_Python
    python3 opencv_gst_test.py
    ```

2. opencv_gst_test.py code
    ```
    import numpy as np
    import cv2 as cv
    import os
    import time

    cap = cv.VideoCapture('v4l2src device=/dev/video1 ! video/x-raw, format=NV12, width=640, height=480, framerate=30/1 ! videoconvert ! appsink', cv.CAP_GSTREAMER)

    if not cap.isOpened():
        print("Cannot capture from camera. Exiting.")
        os._exit()
    last_time = time.time()

    while(True):

        ret, frame = cap.read()
        this_time = time.time()
        print (str((this_time-last_time)*1000)+'ms')
        last_time = this_time;
        cv.imshow('frame', frame)

        if cv.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv.destroyAllWindows()
    ```
## USB Ethernet

USB Ethernet, the main realization is to use the OTG interface of the device as a peripheral mode and simulate it as a network interface, and then the host connects to the device via USB and accesses the Internet through the device. The following is the specific operation based on Firefly-RK3399 device.

Operating environment:

* PC with Ubuntu system
* Firefly-RK3399 device


### Kernel Settings

In the kernel directory, open the kernel configuration options menu:

```
make firefly_linux_defconfig
make menuconfig
```

After entering the kernel configuration menu, select in turn: `Device Drivers` -> `USB Support` -> `USB Gadget Support`.

Set the `USB Gadget Driver` to compile into a module, and then you can find the option of `Ethernet Gadget (with CDC Ethernet support)` below, and choose to compile into a module as well. At the same time, select `RNDIS support`.

```
<M> USB Gadget Drivers
<M> USB functions configurable through configfs
<M> Ethernet Gadget (with CDC Ethernet support)
[*] RNDIS support (NEW)
```

Then compile the kernel in the `kernel` directory:

```
make rk3399-firefly.img -j12
```

After the compilation is completed, burn the kernel to the device. For the burn process, please refer to the **Upgrade Firmware** page of the corresponding board wiki. Then copy the following modules generated in the kernel directory to the device:

* drivers/usb/gadget/function/u_ether.ko
* drivers/usb/gadget/function/usb_f_ecm_subset.ko
* drivers/usb/gadget/function/usb_f_ecm.ko
* drivers/usb/gadget/function/usb_f_rndis.ko
* drivers/usb/gadget/function/usb_f_eem.ko
* drivers/usb/gadget/legacy/g_ether.ko
* drivers/usb/gadget/libcomposite.ko

Then on the device, load the above modules in sequence:

```
insmod libcomposite.ko
insmod u_ether.ko
insmod usb_f_ecm_subset.ko
insmod usb_f_rndis.ko
insmod usb_f_ecm.ko
insmod usb_f_eem.ko
insmod g_ether.ko
```

**Note:** You must load `libcomposite.ko` and `u_ether.ko` first, and then the following modules can be loaded.

### IP address settings

Connect the PC and the OTG interface of the device with a data cable, and execute the `lsusb` command in the PC to view the USB Ethernet device, which means the connection is successful.

```bash
firefly@Desktop:~$ lsusb
Bus 002 Device 003: ID 09da:5814 A4Tech Co., Ltd.
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 005: ID 04f2:b2ea Chicony Electronics Co., Ltd Integrated Camera [ThinkPad]
Bus 001 Device 004: ID 0a5c:21e6 Broadcom Corp. BCM20702 Bluetooth 4.0 [ThinkPad]
Bus 001 Device 003: ID 147e:1002 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

`ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget` in the output information is the USB network card device.

The device is plugged into the network cable so that the device can connect to the external network.

* IP settings in the device:

Enter and execute the `ifconfig -a` command, you can see the following information:

```bash
root@firefly:~# ifconfig -a

# eth0 is the wired network card
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
        inet 168.168.100.48 netmask 255.255.0.0 broadcast 168.168.255.255
        inet6 fe80::1351:ae2f:442e:e436 prefixlen 64 scopeid 0x20<link>
        ether 8a:4f:c3:77:94:ac txqueuelen 1000 (Ethernet)
        RX packets 9759 bytes 897943 (897.9 KB)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 236 bytes 35172 (35.1 KB)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
        device interrupt 42 base 0x8000

        ...

# usb0 is a virtual usb network card
usb0: flags=4098<BROADCAST,MULTICAST> mtu 1500
        ether 4a:81:b1:34:d2:ad txqueuelen 1000 (Ethernet)
        RX packets 0 bytes 0 (0.0 B)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 0 bytes 0 (0.0 B)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

Then customize an appropriate IP for the usb0 network card:

**Note that the IP of usb0 and the IP of wired network card eth0 are not in the same network segment**! !

```
ifconfig usb0 192.168.1.101
```

* IP setting in PC:

```bash
# First check the USB virtual network card
firefly@Desktop:~$ ifconfig
enp0s20u2i1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
        inet 192.168.2.90 netmask 255.255.255.0 broadcast 192.168.2.255
        inet6 fe80::871c:b87e:1327:7fd4 prefixlen 64 scopeid 0x20<link>
        ether 46:fe:6e:97:ee:a6 txqueuelen 1000 (Ethernet)
        RX packets 0 bytes 0 (0.0 B)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 1 bytes 54 (54.0 B)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
...


# Set the USB network card's IP address and the device's usb0 IP address in the same network segment
firefly@Desktop:~$ sudo ifconfig enp0s20u2i1 192.168.1.100

#Set the default gateway: it should be set to the ip address of the device usb0, because the traffic will be forwarded through usb0 later
firefly@Desktop:~$ sudo route add default gw 192.168.1.101
```

After setting the IP of the device and the PC, they can ping each other, and the PC can also use the `ssh` command to log in to the device.

### Network sharing to realize PC Internet access

On the device: First turn on the IPv4 forwarding function:

```bash
echo 1> /proc/sys/net/ipv4/ip_forward
```

If you want to automatically turn on the forwarding function every time you restart the device, please directly modify the value of `net.ipv4.ip_forward` in the `/etc/sysctl.conf` file to 1. After modifying the file parameters, execute the `sysctl -p` command to reload the `/etc/sysctl.conf` file to make the IPv4 forwarding function effective.

Add rules for traffic forwarding:

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 168.168.100.48
```

Now the host PC can access the network. If the PC can ping the usb0 and eth0 of the device, but cannot access the Internet, you need to modify the DNS of the PC and add the following in `/etc/resolv.conf`:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Pay attention to the following points during the configuration process:

* Correspond to each IP address on your device in the above steps, and note that the USB virtual network card IP and wired network IP on the device are not in the same network segment;
* The gateway of the PC virtual USB network card should be set to the IP address of the device virtual USB network card;
* The virtual network card IP address, IP forwarding function, traffic forwarding rules and other settings on the device will be restored after the device is restarted. The user can find out the information on how to automatically set these configurations after booting.
## Network booting

Network booting is to use TFTP to download the kernel and dtb files from the server to the memory of the target machine. At the same time, you can use NFS to mount the network root file system to the target machine to achieve diskless booting of the target machine. The following is an example based on Firefly-RK3399 for user reference.

Ready to work:
* Firefly-RK3399 device;
* Router, network cable;
* Server with NFS and TFTP installed;
* A created root file system.

Note: In the example, the Ubuntu system PC is used as the server, and the connection with the device is realized through a router and a network cable. Users can make adjustments according to their actual conditions, but if the PC is directly connected to the device, please use a crossover cable. Please make sure that the server and the target machine are in the same LAN.

### Server deployment
1.Deploy TFTP service on the server:

Install TFTP service:
```
sudo apt-get install tftpd-hpa
```

Create the `/tftpboot` directory and grant permissions:
```
mkdir /tftpboot
sudo chmod 777 /tftpboot
```

Then modify the configuration file `/etc/default/tftpd-hpa` of the TFTP server, and modify the content as follows:
```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot" #This is set according to the user's actual tftp directory
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-c -s -l"
```

Restart the TFTP server after exiting the modification:
```
sudo service tftpd-hpa restart
```

Create a file in the `/tftpboot` directory to test whether the TFTP service is available:
```bash
firefly@Desktop:~$ cd /tftpboot/
firefly@Desktop:/tftpboot$ touch test
firefly@Desktop:/tftpboot$ cd /tmp
firefly@Desktop:/tmp$ tftp 127.0.0.1
tftp> get test
tftp> q
```
Check the files in the `/tmp` directory. If you see the `test` file, it means the TFTP server is available.

2.Deploy the NFS service on the server:

Install the NFS server:

```
sudo apt-get install nfs-kernel-server
```

Create a shared directory:

```
sudo mkdir /nfs
sudo chmod 777 /nfs
cd /nfs
sudo mkdir rootfs
sudo chmod 777 rootfs
```

Then copy the created root file system to the `/nfs/rootfs` directory.

Then configure NFS and add the location of the shared directory in `/etc/exports`:

```bash
/nfs/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

The shared directory is set according to the actual situation of the user, and the "*" represents that all users can access.

Restart the NFS server:

```
sudo /etc/init.d/nfs-kernel-server restart
```

Mount the shared directory locally to test whether the NFS server is available:

```
sudo mount -t nfs 127.0.0.1:/nfs/rootfs/ /mnt
```

If the contents consistent with `/nfs/rootfs` are viewed in the `/mnt` directory, it indicates that the NFS server is deployed successfully. Then you can unmount:

```
sudo umount /mnt
```

### Kernel configuration

If you want to mount the network root file system, you need to configure the kernel and modify the configuration of the root file system in dts.

First perform the kernel configuration, execute `make menuconfig` in the kernel directory, and select the relevant configuration:

```
[*] Networking support --->
         Networking options --->
                [*] IP: kernel level autoconfiguration
                [*] IP: DHCP support
                [*] IP: BOOTP support
                [*] IP: RARP support


File systems --->
        [*] Network File Systems --->
                [*] Root file system on NFS
```

Modify the `rk3399-firefly.dts` configuration, modify the `bootargs` parameter under the `chosen` node in the dts file, and choose to use NFS to mount the remote root file system. The content is as follows.

```bash
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 root=/dev/nfs rootfstype=ext4 rootwait";
};

# Mainly to modify the value of root, root=/dev/nfs means to mount the network root file system through NFS
```

Compile the kernel:

```
make ARCH=arm64 rk3399-firefly.img -j12
```

After the compilation is complete, copy the compiled kernel files `boot.img` and `rk3399-firefly.dtb` files to the `/tftpboot` directory:

```
cp boot.img /tftpboot
cp arch/arm64/boot/dts/rockchip/rk3399-firefly.dtb /tftpboot
```

For detailed instructions, please refer to `kernel/Documentation/filesystems/nfs/nfsroot.txt` in the kernel directory


### U-Boot Settings

Please make sure that the network cable of the target machine is plugged in and connected to the LAN of the server.

The target machine boots into U-Boot command line mode, and set the following parameters:

```bash
=> setenv ipaddr 192.168.31.101 # Set the IP address of the target machine
=> setenv serverip 192.168.31.106 # Set serverip as the server IP address

# Set to download the kernel and dtb file from TFTP to the corresponding address, users should modify the corresponding address according to their actual target machine
=> setenv bootcmd tftpboot 0x0027f800 boot.img \; tftpboot 0x08300000 rk3399-firefly.dtb \; bootm 0x0027f800-0x08300000

# Set and mount the network root file system, the IP parameters are in order: target machine IP: server IP: gateway: netmask: device name: off, you can set ip=dhcp more simply, and automatically assign IP through DHXP
=> setenv bootargs root=/dev/nfs rw nfsroot=192.168.31.106:/nfs/rootfs,v3 ip=192.168.31.101:192.168.31.106:192.168.31.1:255.255.255.0::eth0:off

# If you don’t want to set it every time you boot, you can save the variables configured above
=> saveenv
Saving Environment to MMC...
Writing to MMC(0)... done

# Start the target machine
=> boot
ethernet@fe300000 Waiting for PHY auto negotiation to complete. done
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename'boot.img'.
Load address: 0x27f800
Loading: ############################################# ################
         ############################################## ###############
         ############################################## ###############
         ############################################## ###############
         ############################################## ###############
         ##############################################
         475.6 KiB/s
done
Bytes transferred = 20072448 (1324800 hex)
Speed: 100, full duplex
Using ethernet@fe300000 device
TFTP from server 192.168.31.106; our IP address is 192.168.31.101
Filename'rk3399-firefly.dtb'.
Load address: 0x8300000
Loading: #######
         645.5 KiB/s
done
Bytes transferred = 97212 (17bbc hex)
## Booting Android Image at 0x0027f800 ...
Kernel load addr 0x00280000 size 19377 KiB
## Flattened Device Tree blob at 08300000
   Booting using the fdt blob at 0x8300000
   XIP Kernel Image ... OK
   Loading Device Tree to 0000000073edc000, end 0000000073ef6bbb ... OK
Adding bank: 0x00200000-0x08400000 (size: 0x08200000)
Adding bank: 0x0a200000-0x80000000 (size: 0x75e00000)
Total: 912260.463 ms

Starting kernel ...

...
```

Visible in the boot kernel log:

```
[12.146297] VFS: Mounted root (nfs filesystem) on device 0:16.
```

It indicates that the network root file system has been mounted.

Precautions

* Ensure that the TFTP server and NFS server are available;
* Make sure that the target machine is plugged into the network cable first and then turned on, and is in the same LAN as the server. If it is directly connected to the target machine and the server, please use a crossover network cable;
* In the kernel configuration, `Root file system on NFS` depends on the `IP: kernel level autoconfiguration` option, please select `IP: kernel level autoconfiguration` first, and then you can find the `Root file system on NFS` option;
* In the U-Boot command line, please confirm the burning address of `boot.img` and the burning address of dtb file. (Hint: In the file structure of `boot.img`, there is a 2k header file at the beginning, and then the kernel. So when TFTP downloads the kernel to the target machine, it must be downloaded to the corresponding kernel address minus 0x0800);
* When setting to mount the remote root file system, `v3` in `nfsroot=192.168.31.106:/nfs/rootfs,v3` represents the NFS version information, please add it to avoid the problem of unsuccessful mounting.
## External storage device rootfs mount

In addition to the root file system used in the internal eMMC, you can also use the root file system of external storage devices, such as SD cards, U disks, etc. The following takes SD card as an example to realize the root file system of mounting external storage device on Firefly-RK3399 device.


### Create a partition on the SD card

Insert an SD card into the PC, and use the `gdisk` tool to separate a partition for mounting the root file system:

```bash
# Users, please use `fdisk -l` to query what the SD card device name is, and use the gdisk command to enter the corresponding device
sudo gdisk /dev/sdb

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):

-------------------------------------------------- -----------------

# Enter? View help information
# Press p to display that the SD card currently has partitions. Since the used SD card has not yet created partitions, no partition information can be found

Command (? for help): p
Disk /dev/sdb: 15278080 sectors, 7.3 GiB
Model: CARD-READER
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 5801AE61-92ED-42FB-A144-E522E8E15827
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 15278046
Partitions will be aligned on 2048-sector boundaries
Total free space is 15278013 sectors (7.3 GiB)

Number Start (sector) End (sector) Size Code Name

-------------------------------------------------- -----------------

# Create a new partition, please create a suitable partition size for the root file system according to your actual situation

Command (? for help): n # Enter n to create a new partition
Partition number (1-128, default 1): 1 # Enter 1 to create the first partition
First sector (34-15278046, default = 2048) or {+-}size{KMGTP}: # Press enter directly, use the default value
Last sector (2048-15278046, default = 15278046) or {+-}size{KMGTP}: +3G # Create partition size of 3G
Current type is'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): # Press Enter to use the default value
Changed type of partition to'Linux filesystem'

-------------------------------------------------- -----------------

# After creation, enter i to view the unique GUID of the partition

Command (? for help): i
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: 6278AF5D-BC6B-4213-BBF6-47D333B5CB53 # Use the unique GUID of the partition, just write down the first 12 digits
First sector: 2048 (at 1024.0 KiB)
Last sector: 6293503 (at 3.0 GiB)
Partition size: 6291456 sectors (3.0 GiB)
Attribute flags: 0000000000000000
Partition name:'Linux filesystem'

-------------------------------------------------- -----------------

# Enter wq to save changes and exit

Command (? for help): wq

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.

```

Format the newly created partition.

```bash
# Note sdb1 represents the device name of the partition corresponding to the SD card, please fill in according to the actual situation
sudo mkfs.ext4 /dev/sdb1

```

After formatting is complete, use the `dd` command to burn the created root file system to the newly created partition of the SD card. (For the customization of the root file system, please refer to *[Ubuntu Rootfs Customization](custom_ubuntu_rootfs.md)*)

```bash
# User please fill in the path of the root file system mirror and the device name of the corresponding partition of the SD card according to the actual situation
dd conv=fsync,notrunc if=/rootfs_path/rootfs.img of=/dev/sdb1
```

### Mount SD card root file system

First, modify the device tree file, open the corresponding dts file, and rewrite the `root` value of the `bootargs` parameter under the `chosen` node under the root node to the unique GUID of the SD card partition written in the root file system:

```bash
# Change the value of root to the first 12 digits of the unique GUID value obtained
chosen {
    bootargs = "earlycon=uart8250,mmio32,0xff1a0000 swiotlb=1 console=ttyFIQ0 rw root=PARTUUID=6278AF5D-BC6B rootfstype=ext4 rootwait";
};

```

Compile and burn to the device after modification.

After the root file system of the SD card has been burned, insert it into the TF card slot of the device and boot into the root file system of the SD card.


Precautions:
* Please pay attention to back up important files in the U disk or SD card before operation to avoid loss;
* Please confirm the device name corresponding to your SD card during operation;
* The value of the `root` parameter in the dts file uses unique GUID, just write down the first 12 digits. The user can also modify this value according to the help information of `gdisk`.
## Update kernel and U-Boot online

This section introduces a simple process of online update. Pack the kernel, U-Boot or other files that need to be updated into a deb installation package, and then import it to the local package warehouse to download and update automatically on the device. For user reference only.

### Prepare deb installation package

The kernel and U-Boot need to be upgraded during the operation, and the modified related files have been prepared in advance: `uboot.img`, `trust.img`, `boot.img`.

deb is the package format of Debian Linux. The most important thing for packaging is to create a `control` file in the `DEBIAN` directory. First create the deb working directory, and then create the corresponding directories and files in the deb directory:

```bash
mkdir deb
cd deb
mkdir firefly-firmware # Create firefly-firmware directory
cd firefly-firmware
mkdir DEBIAN # Create a DEBIAN directory, this directory is required.
mkdir -p usr/share/{kernel,uboot}
mv ~/boot.img ~/deb/firefly-firmware/usr/share/kernel # Place the corresponding file in the corresponding directory
mv ~/uboot.img ~/deb/firefly-firmware/usr/share/uboot
mv ~/trust.img ~/deb/firefly-firmware/usr/share/uboot
```

The files stored in the `DEBIAN` directory are the control files for the deb package installation and the corresponding script files.

Create the control file `control` and the script file `postinst` in the `DEBIAN` directory.

The content of the `control` file is as follows, which is used to record software identification, version number, platform, dependency information and other data.

```bash
Package: firefly-firmware # File directory name
Version: 1.0 # Version number
Architecture: arm64 # Architecture
Maintainer: neg # Maintainer, you can customize
Installed-Size: 1
Section: test
Priority: optional
Descriptionon: This is a deb test
```

The content of the `postinst` file is as follows, which is to write the kernel and U-Boot files that need to be updated into the script of the corresponding partition with the `dd` command:

```bash
echo "-----------uboot updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/uboot.img of=/dev/disk/by-partlabel/uboot

echo "-----------trust updating------------"
dd conv=fsync,notrunc if=/usr/share/uboot/trust.img of=/dev/disk/by-partlabel/trust

echo "-----------kernel updating------------"
dd conv=fsync,notrunc if=/usr/share/kernel/boot.img of=/dev/disk/by-partlabel/boot
```

Description: The `postinst` script is a script that runs after unpacking the data, and the corresponding ones are:

* preinst: a script that runs before unpacking the data;
* prerm: When uninstalling, the script that runs before deleting files;
* postrm: a script to run after deleting the file;

Users can understand other related knowledge points by themselves. Only the `preinst` script is used here.

The following is the created directory tree:

```
deb
└── firefly-firmware
    ├── DEBIAN
    │ ├── control
    │ └── postinst
    └── usr
        └── share
            ├── kernel
            │ └── boot.img
            └── uboot
                ├── trust.img
                └── uboot.img
```

Enter the deb directory and use the `dpkg` command to generate the deb package:

```
dpkg -b firefly-firmware firefly-firmware_1.0_arm64.deb
```

When generating a deb package, it is generally named according to this specification: `package_version-reversion_arch.deb`.

### Create a local warehouse

First install the required packages:

```
sudo apt-get install reprepro gnupg
```

Then use the GnuPG tool to generate a GPG key. After executing the command, follow the prompts:

```
gpg --gen-key
```

Execute `sudo gpg --list-keys` to view the key information just generated:

```
sudo gpg --list-keys

gpg: WARNING: unsafe ownership on homedir'/home/firefly/.gnupg'
gpg: checking trust database
gpg: marginals needed: 3 completes needed: 1 trust model: pgp
gpg: Depth: 0 Validity: 4 Signed: 0 Trust: 0-, 0q, 0n, 0m, 0f, 4u
gpg: The next trust database check will be conducted on May 29, 2021
/home/firefly/.gnupg/pubring.kbx
--------------------------------
pub rsa3072 2019-05-31 [SC] [Valid until: 2021-05-30]
      BCB65788541D632C057E696B8CBC526C05417B76
uid [absolutely] firefly <firefly@t-chip.com>
sub rsa3072 2019-05-31 [E] [Valid until: 2021-05-30]
```

Next to create a package warehouse, first create a directory:

```bash
cd /var/www
mkdir apt # package warehouse directory
mkdir -p ./apt/incoming
mkdir -p ./apt/conf
mkdir -p ./apt/key
```

Export the previously generated key to the warehouse folder, and please correspond to the user name and email address you created.

```
gpg --armor --export firefly firefly@t-chip.com> /var/www/apt/key/deb.gpg.key
```

Create a `distributions` file in the `conf` directory with the following content:

```bash
Origin: Neg # Your name
Label: Mian # The name of the library
Suite: stable # (stable or unstable)
Codename: bionic # Release code name, can be customized
Version: 1.0 # release version
Architectures: arm64 # Architecture
Components: main # component name, such as main, universe, etc.
Description: Deb source test # Related description
SignWith: BCB65788541D632C057E696B8CBC526C05417B76 # The GPG key generated in the above step
```

Build a warehouse tree:

```
reprepro --ask-passphrase -Vb /var/www/apt export
```

Add the deb package made in step 2 to the warehouse:

```
reprepro --ask-passphrase -Vb /var/www/apt includedeb bionic ~/deb/firefly-firmware_1.0_arm64.deb
```

You can view the files added in the library:

```bash
root@Desktop:~# reprepro -b /var/www/apt/ list bionic
bionic|main|arm64: firefly-firmware 1.0
```

Your package has been added to the repository, if you want to remove it, use the following command:

```
reprepro --ask-passphrase -Vb /var/www/apt remove bionic firefly-firmware
```

Install nginx server:

```
sudo apt-get install nginx
```

Modify the nginx configuration file `/etc/nginx/sites-available/default` to:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/apt; //The path of the local package warehouse

    access_log /var/log/nginx/repo.access.log;
    error_log /var/log/nginx/repo.error.log;

    location ~ /(db|conf) {
        deny all;
        return 404;
    }
}
```

Restart the nginx server:

```
sudo service nginx restart
```

### Client update installation

In the client device, first add the source of the local package repository, add a new configuration file `bionic.list` under the directory `/etc/apt/sources.list.d`, the content is as follows:

```
deb http://192.168.31.106 bionic main
```

The IP address is the server address, `bionic` is the warehouse release code name, and `main` is the component name.

Obtain and add the GPG key from the server:

```
wget -O-http://192.168.31.106/key/deb.gpg.key | apt-key add-
```

After the update, you can install the `firefly-firmware_1.0_arm64` package in the custom software source:

```bash
root@firefly:/home/firefly# apt-get update
Hit:1 http://192.168.31.106 bionic InRelease
Hit:2 http://wiki.t-firefly.com/firefly-rk3399-repo bionic InRelease
Hit:3 http://ports.ubuntu.com/ubuntu-ports bionic InRelease
Hit:4 http://archive.canonical.com/ubuntu bionic InRelease
Hit:5 http://ports.ubuntu.com/ubuntu-ports bionic-updates InRelease
Hit:6 http://ports.ubuntu.com/ubuntu-ports bionic-backports InRelease
Hit:7 http://ports.ubuntu.com/ubuntu-ports bionic-security InRelease
Reading package lists... Done

root@firefly:/home/firefly# apt-get install firefly-firmware
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libllvm6.0
Use'apt autoremove' to remove it.
The following NEW packages will be installed:
  firefly-firmware
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 0 B/6982 kB of archives.
After this operation, 1024 B of additional disk space will be used.
Selecting previously unselected package firefly-firmware.
(Reading database ... 117088 files and directories currently installed.)
Preparing to unpack .../firefly-firmware_1.0_arm64.deb ...
Unpacking firefly-firmware (1.0) ...
Setting up firefly-firmware (1.0) ...
-----------uboot updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.437281 s, 9.6 MB/s
-----------trust updating------------
8192+0 records in
8192+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.565762 s, 7.4 MB/s
-----------kernel updating------------
39752+0 records in
39752+0 records out
20353024 bytes (20 MB, 19 MiB) copied, 0.1702 s, 120 MB/s
```

You can see that the poinst script in the deb package was executed during the installation process. After the installation is complete, restart the device and the update is complete.

In the `/usr/share` directory, you can also see the `kernel` and `uboot` directories, which store `boot.img`, `uboot.img` and `trust.img` respectively.

Precautions

* In making the deb package, the directory at the same level as `DEBIAN` is regarded as the root directory, that is, the files placed in the same directory as `DEBIAN`, after the deb package is installed on the client, it can be found in the root directory of the client;
* The files and scripts in the deb package should be adjusted according to their actual situation;
* Every time you modify the configuration file in the warehouse, you must re-import the warehouse directory tree;
* In the nginx server configuration, the `root` parameter configures the address of the warehouse, please modify it according to your actual situation;
* When the client adds the new download source file, please check the correct server address, package warehouse code name and component name. Note that the client must connect to the server;
* The client must use the `apt-key add` command to add the GPG key before it can obtain the local warehouse information.


