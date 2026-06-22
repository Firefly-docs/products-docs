# Quick Use

This chapter describes how to use the official baseboard of `CORE-1126-JD4/CORE-1109-JD4`, and does not include specific application development.

## Power On

The device comes with AI-IPC scene firmware by default.
1. Connect to `DC 12V` power supply, or `POE` power supply.
2. After powering on, the two LED lights next to Ethernet will be on.
3. Connect the device to the network cable and make sure it is in the same LAN as the `windows` computer used for debugging. The computer and 1126 board are required to be in the same network segment.
4. Download [RK_IPCamera_Tool-Vx.x.zip](http://www.t-firefly.com/doc/download/101.html) on windows, unzip and run the software.
5. Use the software to search for the IP of the AI ​​network camera device.
![](../../../rv1126b_img/AIO-1126BJD4V0/iptool.png)
6. Enter the device IP address on the browser to enter the management page. The default account and password are both `admin`.
![](../../../rv1126b_img/AIO-1126BJD4V0/login.png)
7. Use the RTSP stream player to preview the camera.
![](../../../rv1126b_img/AIO-1126BJD4V0/vlc.png)
8. If an external MIPI display is connected, the camera preview will appear on the display.

## Device Debugging

The device can be debugged using `USB cable`, `network port` or `serial port`.

### USB Cable Debugging

#### Hardware Wiring

1. Prepare a male-to-male USB cable
![](../../../rv1126b_img/AIO-1126BJD4V0/usb.jpeg)
2. Connect one end of the USB cable to the USB port of the computer and the other end to the USB OTG debugging interface of the RV1126/RV1109 baseboard. The wiring is as shown below:
![](../../../rv1126b_img/AIO-1126BJD4V0/usb_connect.jpeg)
3. Install ADB based on your system.

#### Windows 下的 ADB 安装：

#### ADB installation under Windows:

Windows version download link: https://dl.google.com/android/repository/platform-tools-latest-windows.zip

1. Unzip `platform-tools-latest-windows.zip` to a custom directory
2. Press `windows + r` to open Run, enter `sysdm.cpl`, and press Enter. Advanced-->Environment Variables-->System Variables-->Double-click Path-->New
3. Example: Set as shown below:
![](../../../rv1126b_img/AIO-1126BJD4V0/set_path.png)
4. Note: The path needs to be changed to the custom directory you unzipped
5. 以管理员权限打开命令提示符，此时电脑 USB 接口连接了 RV1126 的 USB 调试口。命令提示符终端输入 `adb devices` 
   ```
   C:\Users\lvsx>adb devices
   * daemon not running; starting now at tcp:5037
   * daemon started successfully
   List of devices attached
   d70a14329414b400        device
   ```
6. 使用 ADB 进行调试。
   ```
   adb shell
   ```

#### ADB installation under Ubuntu
1. Install android-tools-adb package with apt-get
   ```
   sudo apt-get update
   sudo apt-get install android-tools-adb
   ```

2. Verify installation:
   ```
   lvsx@lvsx:~$ adb devices
   * daemon not running; starting now at tcp:5037
   * daemon started successfully
   List of devices attached
   d70a14329414b400	device
   ```
3. Use ADB to enter device debugging.
   ```
   adb shell
   ```

4. Use ADB to copy the linux pc file to the userdata directory of the rv1126 device.
   ```
   adb push /path/to/your-file /userdata/
   ```

5. Use ADB to copy the rv1126 device file to the local directory of the PC. Replace rv1126-file with the actual file name.
   ```
   adb pull /path/to/rv1126-file ./
   ```

### Network Port Debugging

View the device IP through ` RK_IPCamera_Tool `. The RK_IPCamera_Tool tool can be downloaded from the resource download page.

#### Network ADB Debugging
1. Use a network cable to access any network interface
2. Connect the network cable to the device network ADB.
   ```
   adb connect 192.168.22.33
   ```
3. Use ADB for debugging.
   ```
   adb shell
   ```

#### SSH Debugging

1. ssh to log in to the development board for debugging

2. The default account of the firmware before September 2021 is: root, and the default password is: rockchip.

3. The default account of the firmware after September 2021 is: root, and the default password is: firefly.
   ```
   ssh root@192.168.22.33
   ```

* The default account of Debian10 system is: firefly, and the default password is: firefly.
   ```
   ssh firefly@192.168.22.33
   ```

### Serial Port Debugging

1. Prepare a USB to serial port module. It is recommended to use the serial port module of Firefly.

2. Install the serial port module.

3. The USB serial port module connects the host USB port and the device serial port.
4. Set the baud rate to `1.5M`.
5. Download, install, and open the serial port debugging tool supported by your system platform (minicom is recommended for Ubuntu and putty is recommended for Windows). If the serial port debugging tool has configuration items for software and hardware flow control, please turn off this function.
6. The hardware connection of Firefly's serial port module is shown in the figure:

![](../../../rv1126b_img/AIO-1126BJD4V0/DEBUG.jpg)