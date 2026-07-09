# First use

## Product Overview

iCore-1684XQ The core board is equipped with the SOPHON AI processing processor BM1684X, which can be configured with 16GB of large memory and 64GB of eMMC storage. It supports 32TOPS (INT8) peak performance, 16TFLOPS (FP16/BF16) performance, and 2TFLOPS (FP32) high precision performance. It also supports 32-channel H.265/H.264 1080p@25fps video decoding.

The core board features a BTB interface with PCIe 3.0, GMAC, SDIO 3.0, I2C, PWM, UART, and GPIO interfaces, making it easy to integrate into various edge embedded products. It supports mainstream programming frameworks and comes with a comprehensive toolchain for easy development, with minimal migration costs for algorithm migration.

The provided reference design material for the baseboard enables users to customize it deeply according to their needs. It efficiently adapts to all AI algorithms available in the market, empowering industries such as visual computing, edge computing, general computing services, artificial intelligence, and industries like smart construction, transportation, education, retail, surveillance, and security with AI capabilities.

**Front View of iCore-1684XQ Core Board:**

![](../../../bm1684_img/AIO-1684XQ/iCore-1684XQ_front.jpg)

**Back View of iCore-1684XQ Core Board:**

![](../../../bm1684_img/AIO-1684XQ/iCore-1684XQ_back.jpg)

AIO-1684XQ The motherboard consists of a core board AIO-1684XQ + baseboard SM7-SBASE-EN4. The AIO-1684XQ comes with onboard interfaces including M.2 (5G), Mini PCIe, SATA 3.0, Gigabit Ethernet, USB 3.0, RS485, RELAY, GPIO, and other rich interfaces.

![](../../../bm1684_img/AIO-1684XQ/AIO-1684XQ.jpg)

**Product Specifications:**

![](../../../bm1684_img/AIO-1684XQ/parameter.png)

## Accessory List

Upon receiving the development board, ensure all the accessories are included:
- If a cased unit, make sure it contains the integrated board, or if not, ensure you have the iCore-1684XQ Core Board and SM7-SBASE-EN4 Baseboard each along with a cooling fan.
- One 12V-5A power adapter.

Optionally purchased accessories include:
- Firefly serial port module.

Additionally, during usage, you might require the following:
- Display device:
    + Monitor or TV with an HDMI interface, along with an HDMI cable.
- Network:
    + 100M/1000M Ethernet cable, wired router.
    + WiFi router.
- Firmware upgrade:
    + TF card, 4GB or larger with a Class 10 rating.
- Debugging:
    + USB to serial adapter.

## Remote Network Login

![](../../../bm1684_img/AIO-1684XQ/ethernet_interfaces.png)

Ethernet port 0 (located on the upper layer) is set up with a dynamic IP, which can be obtained automatically via DHCP.  

Ethernet port 1 (located on the lower layer) is configured with a static IP 192.168.150.1 and subnet mask 255.255.255.0. You can set your PC's IP address to 192.168.150.2/24 for initial access.

If your PC can successfully ping the IP address of the Ethernet port, you can proceed to SSH login. The SSH port is 22, and the username and password are both linaro:

```
ssh linaro@192.168.150.1
```

Once connected to the development board, you can modify the network configuration. The method is similar to Debian/Ubuntu desktop versions. You can configure it in the /etc/netplan/ directory.

## Serial Terminal Access

Refer to [Serial Debugging](debug.md).

## Power Off

When shutting down, it's recommended to avoid directly disconnecting the power. First, run `sudo poweroff`, then disconnect the power to avoid damaging the file system data.

Additionally, if you've successfully entered the Linux system, you can also press and hold the power button. The system will detect it and safely shut down the system and the development board's power.