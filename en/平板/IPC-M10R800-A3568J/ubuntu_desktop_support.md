# Firefly Ubuntu Desktop Introduction

IPC-M10R800-A3568J The Ubuntu Desktop system is based on the Ubuntu 64bit system. Currently, we mainly maintain Ubuntu 20.04. Has the following characteristics:

1. The desktop environment uses LXQT or XFCE4, Firefly custom theme, simple and beautiful.
1. Xserver uses GPU + RGA for 2D acceleration, runs smoothly, and takes up less CPU resources.
1. For embedded platforms, streamline system services.
1. Provide OpenGL and OpenCL support based on Arm Mali GPU.
1. Provide video hard codec support based on Rockchip VPU + Mpp.
1. Adapt to QT, Docker and other development frameworks.
1. Provide a series of interfaces to operate onboard resource devices.
1. The system adopts overlayfs file system, supports exporting rootfs, secondary packaging, restoring factory settings and other functions


## basic information

### User and password

After the Firefly Linux Desktop system boots up, it will automatically log in to the firefly user.

If there is a debugging serial port connected, the serial terminal automatically logs in to the root user.

* Firefly user password: firefly

* Root user: No root password is set by default. Firefly users configure the root password by themselves through the `sudo passwd root` command.

For more introduction, please refer to ["Firefly Ubuntu User Manual"](ubuntu_manual.md)