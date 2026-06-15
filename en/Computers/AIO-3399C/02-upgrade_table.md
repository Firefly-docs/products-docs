# Instructions for writing with USB cable (important)


## Preface

AIO-3399C is installed with Android7.1(industry) system by default. 

AIO-3399C may officially support multiple systems, so it is possible to upgrade firmware for different systems.

The firmware of each system has its own version of the upgrade tool. If the version of the upgrade tool is incorrect, the firmware may fail to be upgraded. In particular, pay attention to the **cross-system upgrade** scenario.

This chapter mainly introduces **The relationship between system firmware and upgrade tool version** and **Considerations for cross-system upgrade**.

## The relationship between system firmware and upgrade tool version

| system firmware | upgrade tool |
| :----:| :----: |
| Android7.1(tvbox) | Windows: **AndroidTool v2.38** <br/> Linux: **upgradetool v1.24** |
| Android7.1(industry) | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android8.1 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android10.0 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.49** |
| Ubuntu | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |

Download the corresponding version of the upgrade tool according to the table above:

* [Linux_Upgrade_Tool](http://en.t-firefly.com/doc/download/52.html#other_367)
* [Androidtool_xxx](http://en.t-firefly.com/doc/download/52.html#other_343)

## Considerations for cross-system upgrade
### Explanation of noun

`cross-system upgrade`: The system firmware to be upgraded and the current system firmware are not part of the same system firmware.

### Notes

Cross-system upgrade firmware generally needs to be **erased first**, and **then upgraded**. In addition, **the corresponding version of the upgrade tool must be used** for both erase and burn. The steps are as follows:

* Use the corresponding version of the upgrade tool to the current system firmware to erase
* Use the corresponding version of the upgrade tool to the system firmware to upgrade

#### For example

AIO-3399C upgraded Android7.1(industry) firmware by default when it is delivered, and the customer needs to upgrade Android10.0 firmware for example:

According to [The relationship between system firmware and upgrade tool version](02-upgrade_table.html#the-relationship-between-system-firmware-and-upgrade-tool-version), it can be known that:

| system firmware | upgrade tool |
| :----:| :----: |
| Android7.1(industry) | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android10.0 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.49** |

The steps are as follows:

* Use **AndroidTool v2.65** or **upgradetool v1.34** to erase
* Use **AndroidTool v2.65** or **upgradetool v1.49** to upgrade



[Linux_Upgrade_Tool]: http://en.t-firefly.com/doc/download/52.html#linux_12
[upgrade_tool_xxx (version number)]: http://en.t-firefly.com/doc/download/52.html#linux_12
[RK3399_Android7.1_EraseIDB.7z]: http://en.t-firefly.com/doc/download/52.html#windows_341