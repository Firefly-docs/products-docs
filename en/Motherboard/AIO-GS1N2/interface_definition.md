# Interface 
## Enclosure
![](../../../gs1-n2_img/GS1-N2/interface-enclosure-en.png)

## Main Board
![](../../../gs1-n2_img/GS1-N2/interface-mainboard-en.png)

|Main mod|When only installing the main mod|
|----|----|
|RK3588(Core-3588JD4)|1. SATA0, PCIe0, PCIe WiFi(M.2) : Choose one of three<br>2. PCIe1(PCIe 3.0), Connect the sub mod: Choose one of two<br>3. SATA2 available (dedicated to the main mod Core-3588JD4, not available for other modules)<br>4. Other PCIe/SATA are not available|
|RK3576(Core-3576JD4)|1. SATA0, PCIe0, PCIe WiFi(M.2) : Choose one of three<br>2.  SATA1, PCIe1: Choose one of two<br>3.  Other PCIe/SATA are not available|
|SOPHGO(Core-1688JD4)|1.  SATA0, PCIe0, PCIe WiFi(M.2) : Choose one of three<br>2. PCIe1: available<br>3. Other PCIe/SATA are not available|
|NVIDIA(Jetson Orin Nano)<br>NVIDIA(Jetson Orin NX)|1. PCIe0, PCIe WiFi(M.2) : Choose one of two<br>2. PCIe1: available<br>3. TF Card, management network port: not available<br>4.  Other PCIe/SATA are not available|

|Sub mod|When installing the sub mod|
|----|----|
|1.  You can choose anyone of the main mod<br>2. The sub mod can be empty|1. RES_PCIe are available<br>2. PCIe2 is only available if the sub mod is:<br>&ensp;&ensp;Core-3588JD4 / Jetson Orin Nano / Jetson Orin NX|