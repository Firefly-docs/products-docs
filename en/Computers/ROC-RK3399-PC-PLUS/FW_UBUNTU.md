


## 2022-01-01

* **Firmware:**
  * ROC-RK3399-PC-PLUS-EDP-UBUNTU-18.04_DESKTOP-GPT-20211228-1700.img.7z
  * ROC-RK3399-PC-PLUS-EDP-UBUNTU-18.04_MINIMAL-GPT-20211229-1344.img.7z
  * ROC-RK3399-PC-PLUS-EDP-UBUNTU-20.04_DESKTOP-GPT-20211229-1344.img.7z
  * ROC-RK3399-PC-PLUS-EDP-UBUNTU-20.04_MINIMAL-GPT-20211229-1345.img.7z
  * ROC-RK3399-PC-PLUS-EXP-UBUNTU-18.04_MINIMAL-GPT-20211229-1345.img.7z
  * ROC-RK3399-PC-PLUS-EXP-UBUNTU-20.04_DESKTOP-GPT-20211229-1345.img.7z
  * ROC-RK3399-PC-PLUS-EXP-UBUNTU-20.04_MINIMAL-GPT-20211229-1346.img.7z
  * ROC-RK3399-PC-PLUS-UBUNTU-18.04_DESKTOP-GPT-20211228-1710.img.7z
  * ROC-RK3399-PC-PLUS-UBUNTU-18.04_MINIMAL-GPT-20211229-1347.img.7z
  * ROC-RK3399-PC-PLUS-UBUNTU-20.04_DESKTOP-GPT-20211229-1348.img.7z
  * ROC-RK3399-PC-PLUS-UBUNTU-20.04_MINIMAL-GPT-20211229-1348.img.7z
* **Kernel commit:** 362d761ebcecd5b73fd6ef81c5a92ad1e67b947b
* **FS:**
    * ubuntu_18.04_RK3399_ext4_v2.10-62-g087b2b2_20211228-1005_DESKTOP.img
    * ubuntu_18.04_RK3399_ext4_v2.10-62-g087b2b2_20211229-1105_MINIMAL.img
    * ubuntu_20.04_RK3399_ext4_v2.10-62-g087b2b2_20211228-1443_DESKTOP.img
    * ubuntu_20.04_RK3399_ext4_v2.10-62-g087b2b2_20211229-1158_MINIMAL.img
* <font color = "red">**Update content:**</font>
  * **Kernel:**
    1. Update bcmdhd version 100.10.545.19 and support AP6398SV wifi.
    2. Adjust the drive current of the SDIO bus.
    3. rk3399-firefly-aio-cam-8ms1m.dts support xc7160.
    4. firefly_linux_defconfig: Enable more gadget functions.
    5. And other.
  * **FS:**
    1. Support AP6398SV wifi.
    2. Bt_fwname support armhf.
    3. Ec20: quectel-CM: update to v1.6.1.
    4. Set power-key to call up the menu.
    5. Update glmark2 icon.
    6. Add glmark2-es-wayland when wayland enabled
    7. Relink mali so when display server switch




## 2021-11-28

* **Firmware:**
    * ROC-RK3399-PC-PLUS-EDP-UBUNTU-18.04_DESKTOP-GPT-20211028-1338.img.7z
    * ROC-RK3399-PC-PLUS-EDP-UBUNTU-18.04_MINIMAL-GPT-20211101-1817.img.7z
    * ROC-RK3399-PC-PLUS-EXP-UBUNTU-18.04_DESKTOP-GPT-20211028-1340.img.7z
    * ROC-RK3399-PC-PLUS-EXP-UBUNTU-18.04_MINIMAL-GPT-20211101-1817.img.7z
    * ROC-RK3399-PC-PLUS-UBUNTU-18.04_DESKTOP-GPT-20211028-1342.img.7z
    * ROC-RK3399-PC-PLUS-UBUNTU-18.04_MINIMAL-GPT-20211101-1818.img.7z
* **Kernel commit:** 84223e13aa4344080ca3adeba1f2e7dd45951010
* **FS:**
    * ubuntu_18.04_RK3399_ext4_v2.10-55-gdb3f844_20211030-1346_MINIMAL.img
    * ubuntu_18.04_RK3399_ext4_v2.10-55-gdb3f844_20211027-1728_DESKTOP.img
* <font color = "red">**Update content:**</font>
  * **Kernel:**
    1. Add HDMI-IN to support GStreamer's 60fps capture.
    2. Remove the console-setup in initrd to make the logo display longer.
    3. rk3399-firefly-CS-R1-jd4-main.dts: Increase the maximun speed limit of pcie0.
    4. rtc-rk808: Remove the time initialization of the probe phaset to ensure the accuracy of rtc.
    5. firefly_linux_defconfig: Enable more gadget functions
    6. And other.
  * **FS:**
    1. Other.



## 2021-04-12

* **Firmware:**
    * ROC-RK3399-PC-PLUS-EDP-UBUNTU-RK3399_UBUNTU_18.04_DESKTOP-GPT-20210408-1817.img.7z
    * ROC-RK3399-PC-PLUS-EXP-UBUNTU-RK3399_UBUNTU_18.04_DESKTOP-GPT-20210408-1818.img.7z
    * ROC-RK3399-PC-PLUS-UBUNTU-RK3399_UBUNTU_18.04_DESKTOP-GPT-20210408-1821.img.7z
* **Kernel commit:** 84223e13aa4344080ca3adeba1f2e7dd45951010
* **FS:**
    * ubuntu_18.04_arm64_ext4_v2.04-1-g618089a_20210316-1035_DESKTOP.img
* <font color = "red">**Update content:**</font>
  * **Kernel:**
    1. Update.
  * **FS:**
    1. Fix the problem that the delay time of the network gmac node does not match.
    2. Fix the file system audio and video dependency problem.
    3. Fix some problems with the camera.


