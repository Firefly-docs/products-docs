# How to use Firefly Ubuntu minimal

CQ38W-3576 The Ubuntu MINIMAL system is based on the Ubuntu 64bit system. Currently, Ubuntu 18.04 is released. Compared with Ubuntu Desktop, it has the following features:

1. No desktop environment, less resources, after simplifying network management, only 40M memory;
1. For embedded platforms, streamline system services.
1. Adapt to QT, Docker, Electron and other development frameworks.
1. Provide a series of interfaces to operate onboard resource devices.
1. The system adopts overlayfs file system, supports exporting rootfs, secondary packaging, restoring factory settings and other functions

## How to use Qt
1. After the Firefly Linux Minimal system is booted, it will automatically log in to the root user with the password firefly
2. The system has added OpenGL ES, OpenCL, DRM support.
3. Due to security policy, sshd does not support remote login via root user by default, it can be turned on by the following method

```
sed -i -e 's/#PermitRootLogin.*/PermitRootLogin yes/g' /etc/ssh/sshd_config
```

For more introduction, please refer to ["Firefly Ubuntu User Manual"](linux_ubuntu_manual.md)