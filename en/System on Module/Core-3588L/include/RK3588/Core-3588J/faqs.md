## Selection of power adapter
At present, the official power supply for ITX-3588J is 12V/7A. If it is a self adapted power supply, it cannot be lower than 12V/7A/80W under full load, otherwise there may be insufficient power supply.

## About the Use of DIP Switches 

DIP switch position:

![](../../../rk3588_img/Core-3588L/faqs_dip_switch.jpg)

-   When the DIP switch is `ON`, the power up and down of the device
    depends on the insertion and removal of the power adapter. 
-   When the dial switch is `OFF`, the power-on of the equipment can be
    realized only after the power adapter is connected and the external Power
    button is pressed. 
-   The DIP switch defaults to the `ON` state. 

## Customized base plate faq
There are many uncertain issues with self-made base plates. When there is a problem of not being able to boot the machine, the following troubleshooting can be carried out
* Whether the power supply of the core board is normal;
* Tailor the functions in the device tree based on the actual hardware usage (such as PCIe, PCA9555, etc.);
* Reference comparison rk3588-firefly-itx-3588j_custom_demo.dtsi is cropped (only enable eth0, eth1, and hdmi0);