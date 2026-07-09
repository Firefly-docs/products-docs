# Instruction of upgrade

Due to differences in firmware of different systems, there may be some problems when upgrade firmware. If you encounter some problems, you can follow the steps below.

## Ready to Work

### Firmware

* **Linux**

    The Linux firmware released by Firefly uses the GPT partition format to manage the boot media. <font color=#ff0000>The early firmware that uses the MBR partition format has been discontinued.</font>

* **Android 5.1**

### Tools

* **[Upgrade_tool](http://en.t-firefly.com/doc/download/47.html#linux_22)**
* **[Android_tool](http://en.t-firefly.com/doc/download/47.html#windows_22)**

## Instruction of upgrade Android5.1

When upgrading Android5.1 firmware, please read table carefully:

![](../../../rk3288_img/load_android_notice.png)

Focus on: In Windows, When Linux upgrade to the Android5.1, how to enter Maskrom Mode:

**Steps are as follows:**

1. In AndroidTool_v2.58, click "Erase Flash", this step is erase IDB, it can't erase flash, so it will appear "erase flash failed";
2. Reboot, in AndroidTool_v2.35, click "Erase Flash", when it finish, the board will enter Maskrom Mode;
3. Click "Upgrade" to start burn the Android5.1 firmware.

**Tips: If click "Erase Flash" or "Upgrade" will appear "test Device failed", change to AndroidTool_v2.47.**

## Instruction of upgrade Linux-GPT

When upgrading Linux-GPT firmware, please read table carefully:

![](../../../rk3288_img/load_linux_gpt_notice.png)

Focus on:

In Windows, Linux-MBR or Android5.1 upgrade to the Linux-GPT, how to enter Maskrom Mode:

**Steps are as follows:**

1. In AndroidTool_v2.58, click "Erase Flash", when it finish, the board will enter Maskrom Mode;
2. Click "Upgrade" to burn Linux-GPT firmware;

In Linux, Linux-MBR or Android5.1 upgrade to the Linux-GPT, how to enter Maskrom Mode:

**Steps are as follows:**

```bash
# In upgrade_tool_v1.24: erase flash with upgrade_tool_v1.24, then the board enter Maskrom Mode;
sudo upgrade_tool_v1.24 ef /path/to/Linux-GPT

# In upgrade_tool_v1.34: upgrade firmware with upgrade_tool_v1.34;
sudo upgrade_tool_v1.34 uf /path/to/Linux-GPT
```

## Instruction of upgrade Linux-MBR

When upgrading Linux-MBR firmware, please read table carefully:

![](../../../rk3288_img/load_linux_mbr_notice.png)

In the above table, they are normal upgrade in Loader Mode.