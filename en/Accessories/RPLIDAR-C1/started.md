# RPLIDAR C1 LiDAR

## 1. Product Introduction

### Overview

RPLIDAR C1 is a hybrid DTOF LiDAR developed by Slamtec for developers and indoor applications. It supports 360° scanning, with a 5 kHz sampling rate and an 8-12 Hz scanning frequency, making it suitable for robot navigation, obstacle avoidance, localization, mapping, and other applications.

![](../../../modules_img/RPLIDAR-C1/rplidar-c1.png)

### Specifications

| Item | Specification |
| ---- | ---- |
| Model | RPLIDAR C1 |
| Type | Hybrid DTOF LiDAR |
| Application | Indoor |
| Mechanical dimensions | 55.6 x 55.6 x 41.3 mm |
| Measurement range | White objects: 0.05-12 m (70% reflectivity); black objects: 0.05-6 m (10% reflectivity) |
| Sampling rate | 5 kHz |
| Scanning frequency | 8-12 Hz, typically 10 Hz |
| Angular resolution | 0.72° |
| Range resolution | 15 mm |
| Range accuracy | ±30 mm |
| Protection rating | IP54 |
| Ambient light immunity | 40 klux |
| Supply voltage | 5 V |
| Data communication interface | TTL UART, 460800 bps |
| Operating temperature | 0°C-50°C |

## 2. Usage

### SDK Usage

The latest SDK is available from GitHub:

```bash
git clone https://github.com/Slamtec/rplidar_sdk.git
cd rplidar_sdk
make
cd output/Linux/Release
ls
```

After compilation, the following files should be available:

```text
custom_baudrate  libsl_lidar_sdk.a  simple_grabber  ultra_simple
```

After connecting the RPLIDAR C1, use the following commands to identify the device node:

```bash
dmesg | grep -E "ttyUSB|cp210x"
ls /dev/ttyUSB*
```

On Firefly Ubuntu platforms, the device node is usually `/dev/ttyUSB0`, and the C1 communication baud rate is `460800`.

#### ultra_simple

`ultra_simple` reads and prints real-time scan data through the serial port:

```bash
./ultra_simple --channel --serial /dev/ttyUSB0 460800
```

Example output:

```text
theta: 0.10 Dist: 00666.00 Q: 47
theta: 0.98 Dist: 00669.00 Q: 47
theta: 1.86 Dist: 00671.00 Q: 47
```

In the output:

* `theta`: Scan angle in degrees, from 0° to 360°.
* `Dist`: Distance to the obstacle in millimeters.
* `Q`: Signal quality. Higher values indicate a stronger signal, while `0` indicates an invalid point.

#### simple_grabber

`simple_grabber` displays scan results from 0° to 360° as ASCII art in the terminal:

```bash
./simple_grabber --channel --serial /dev/ttyUSB0 460800
```

The LiDAR origin is located at the bottom of the ASCII diagram. Denser characters generally indicate that obstacles are closer in the corresponding direction.

#### custom_baudrate

`custom_baudrate` tests communication at a specified baud rate:

```bash
./custom_baudrate /dev/ttyUSB0 460800
```

### ROS 2 Usage

The test platform is Firefly ROC-RK3588S-PC running Ubuntu 22.04 and ROS 2 Humble. For instructions on installing ROS 2 Humble, see the [Firefly ROS 2 Installation Guide](https://community.t-firefly.com/en/docs/software/industry-robot/ROS2).

Download and build the SLAMTEC ROS 2 driver:

```bash
git clone -b ros2 --single-branch https://github.com/Slamtec/rplidar_ros.git
sudo apt install -y python3-colcon-common-extensions
cd rplidar_ros
source /opt/ros/humble/setup.bash
colcon build --symlink-install --parallel-workers 2
source ./install/setup.bash
```

After connecting the LiDAR, confirm that the serial device is `/dev/ttyUSB0`, then start the driver node:

```bash
sudo chmod 777 /dev/ttyUSB0
ros2 launch rplidar_ros rplidar_c1_launch.py
```

Open another terminal, load the ROS 2 and driver environments, and run the client:

```bash
source /opt/ros/humble/setup.bash
cd rplidar_ros
source ./install/setup.bash
ros2 run rplidar_ros rplidar_client
```

The client subscribes to the `/scan` topic and outputs scan data. Angles are in degrees, and distances are in meters.

You can also run the visualization tool `rviz2` to view the point cloud:

```bash
source /opt/ros/humble/setup.bash
cd rplidar_ros
rviz2 -d rviz/rplidar_ros.rviz
```

## 3. Resources

* [RPLIDAR C1 Product Introduction](https://www.slamtec.com/cn/c1)
* [RPLIDAR C1 Documentation and SDK](https://www.slamtec.com/cn/support#rplidar-c1)
* [RPLIDAR SDK](https://github.com/Slamtec/rplidar_sdk)
* [RPLIDAR ROS 2 Driver](https://github.com/Slamtec/rplidar_ros)