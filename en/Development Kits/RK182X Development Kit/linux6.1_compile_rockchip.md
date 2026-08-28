# Main Module

## Download the SDK

Depending on the main module, contact `sales@t-firefly.com` to obtain the **RK3588 Kernel 6.1 SDK** or **RK3576 Kernel 6.1 SDK**, then read the `README` included with the SDK.

<font color=red>

**Notice:**
<br>
**1.** The SDK uses cross-compilation. Use it on an `x86_64` PC; do not download the SDK to the development kit.<br>
**2.** Ubuntu 20.04 (a physical PC or Docker container) is recommended for building. Other operating systems may cause build failures.<br>
**3.** Do not place or extract the SDK in a virtual-machine shared folder or a directory whose path contains non-English characters.<br>
**4.** Use a regular user to download and compile the SDK. Root privileges are only needed when installing system packages.

</font>

* RK3588 SDK: update to at least `rk3588/linux6.1_release_v1.3.0e`.
* RK3576 SDK: update to at least `rk3576/linux_release_v1.3.0a`.

The development kit supports Debian 12 and Ubuntu firmware for the following main modules:

* Core-3588JD4
* Core-3588SJD4 AI
* Core-3576JD4

Select the operating system you want to build:

* [Compile Debian firmware](linux6.1_compile_debian.md)
* [Compile Ubuntu firmware](linux6.1_compile_ubuntu.md)

## Export the Main Module Rootfs

For rootfs export, see [Export device rootfs](/docs/tools/development-tool/Rootfs-Export-Tool/ff-export-rootfs).
