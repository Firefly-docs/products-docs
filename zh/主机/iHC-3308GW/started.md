# 简介

[规格书](https://download.t-firefly.com/Spec/Computers/iHC-3308GW_Specification_CN.pdf) | [购买链接](https://store.t-firefly.com/goods.php?id=147) | [下载资料](https://community.t-firefly.com/doc/download/102)

IHC-3308GW 是专为工业环境打造的 4G 智能网关，采用 IoT 专用的四核 64 位处理器 RK3308B；全面支持 4G LTE、NB-IoT、LoRa 通信；拥有双百兆以太网口以及 RS485、CAN、RS232 等控制接口，并支持国家商用密码安全算法；广泛适用于工业 4G 路由、IoT 物联网、自动化系统等工业领域。

<center>

<img alt="" src="../../../rk3308_img/IHC-3308GW/ihc-3308gw-side1.png" width="600">
</center>
## 接口描述

**IHC-3308GW** 提供了丰富的接口，主要包括：

* 1 x DC IN（12V 电源接口）
* 1 x Type-C（OTG）
* 1 x USB 2.0
* 2 x RJ45（100Mbps 百兆以太网口）
* 1 x SIM 卡槽（4G）
* 1 x PSAM 卡槽
* 1 x CAN
* 1 x RS232
* 3 x RS485
* 1 x DIN（光耦隔离输入）
* 1 x DOUT（继电器输出）
* 1 x Phone（耳机接口）
* 1 x TF-Card
* WiFi 天线（支持 2.4GHz WiFi，802.11 b/g/n）；4G 版本另有 4G 天线

整机接口如下图：

<center>

<img alt="" src="../../../rk3308_img/IHC-3308GW/ihc-3308gw-side1.png" width="800">
</center>
## 结构尺寸

整机尺寸 99.4 mm × 84 mm × 35.2 mm，工作温度 -10℃～60℃，工作湿度 10%～90%。

<center>

<img alt="" src="../../../rk3308_img/IHC-3308GW/size.png" width="800">
</center>

## 产品规格

| 基 本 参 数          |                                                              |
| -------------------- | ------------------------------------------------------------ |
| 主控芯片             | RK3308B（28纳米制程）                                        |
| 处理器               | 四核64位ARM Cortex-A35，主频1.3GHz                           |
| 内存                 | 256M DDR3（128MB ~ 512MB可选）                               |
| 存储器               | 4GB eMMC：支持4G/8G/16G/32G/64G/128G<br>SPI Flash： 支持16MB ~512 MB<br>支持MicroSD (TF) Card Slot扩展 |
| **硬 件 特 性**      |                                                              |
| 以太网               | 双RJ45以太网口（100M bps）                                   |
| WiFi                 | 支持 2.4GHz WiFi，支持802.11/b/g/n协议                       |
| 音频                 | 内置音频CODEC，包含8路ADC，集成高性能Codec和Hardware VAD     |
| 接口                 | PSAM × 1<br>CAN × 1<br>SIM 4G × 1<br>RJ45百兆以太网口 × 2<br>RS232 × 1<br>RS485 × 3<br>DC IN (12V ) × 1<br>Type-C (OTG) × 1<br>DIN × 1<br>DOUT × 1<br>Phone × 1<br>TF-Card × 1<br>USB 2.0 × 1 |
| **系 统 软 件**      |                                                              |
| 系统支持             | 支持Buildroot（Linux）嵌入式系统、Ubuntu 18.04               |
| 无线通信             | 支持4G LTE Cat1无线通信模块，可实现任意运营商的4G网络无缝对接；<br>支持NB-IOT 物联网通信支持全球频段B1/B3/B5/B8/B20/B28等，速度快、功耗低；<br>支持工业级远距离LoRa通信，868MHz频率，户外视距通讯距离高达8Km，高稳定性 |
| MQTT协议             | 支持Modbus标准工业协议转MQTT协议，<br/>云端支持阿里云、私有云部署，适用于物联网PLC数据采集和控制场景 |
| 国家商用密码安全算法 | 板载有PSAM卡卡槽，方便系统集成PSAM卡功能；<br>具有强大的设备认证、数据加密解密功能; <br>提供性能优异的 DES/3DES、 AES、 SHA、 RSA、 ECC；<br>以及国家商用密码 SM1/SM2/SM3/SM4 等安全算法模块 |

## 资源与技术支持

* [[技术交流论坛]](https://forum.t-firefly.com/)：超过10万企业客户和用户沟通交流平台

### 联系方式

* 邮箱：sales@t-firefly.com
* 手机：(+86) 186 8811 7175
* 座机：0760-89881218
* 全国服务热线：4001-511-533
* 地址：广东省中山市东区中山四路 57 号宏宇大厦 2101 室
 <a id="firmware-format"></a>