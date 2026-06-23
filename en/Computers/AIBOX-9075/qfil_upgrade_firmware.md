# Upgrade Complete Firmware

## Enter EDL (Emergency Download) mode

### Hardware Way

1. The device should disconnect all power supply.

2. Use Type-A USB cable to connected device's download port with PC.

3. Use needle or other small tools, insert into recovery hole, press the button in it and hold.

4. Insert the DC power supply.

5. After 2 sec, release the recovery button.

![](../../../qcom_img/AIBOX-9075/edl_key.jpg)
### Software Way

While the device is running normally, connect device's download port with PC through Type-A USB cable, run this command on device terminal or debug console:

```shell
sudo systemctl reboot edl
```

![](../../../qcom_img/AIBOX-9075/download_port.jpg)

### Check EDL

If the device entered EDL mode, QFIL tool will show "Qualcomm HS-USB QDLoader 9008".

![](../../../qcom_img/qfil_9008.jpg)

Or it could show "Please Select an Existing Port", then you have to click "SelectPort", also can see "Qualcomm HS-USB QDLoader 9008", select it and click "OK".

![](../../../qcom_img/qfil_select_port.jpg)

**Notice: The code must be 9008** If the device shows other code like 900E, then it means device is abnormal, please poweroff it and retry.

If tool shows "No Port Available", please check driver installation and usb cable.

## Download Firmware

1 Click "Configuration", then click "FireHose Configuration", in the pop-up window, config it according to the following pictrue, then click "OK".

![](../../../qcom_img/qfil_config.jpg)

2 In the main page, select "Flat Build", then click "Browse"

![](../../../qcom_img/qfil_browse.jpg)

3 Navigate to your firmware location, choose "All Files" in file type, then find "xbl_s_devprg_ns.melf" (or maybe "prog_firehose_ddr.elf", different SOCs very in filenames)and click "open".

![](../../../qcom_img/qfil_open_melf.jpg)

4 In the main page, click "Load XML", in the pop-up window, select all xml files and click "open", then the same window will pop-up again, and select all xml files again and click "open".

![](../../../qcom_img/qfil_load_xml1.jpg)

![](../../../qcom_img/qfil_load_xml2.jpg)

5 Finally click "Download", wait it to finish, it will take few minutes. Download success will be like:

![](../../../qcom_img/qfil_download_finish.jpg)

6 After the download is complete, wait for a while and the device will automatically reboot.