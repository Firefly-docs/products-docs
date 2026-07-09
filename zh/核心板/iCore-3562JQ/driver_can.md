# CAN 使用
## CAN 简介
CAN(Controller Area Network)总线，即控制器局域网总线，是一种有效支持分布式控制或实时控制的串行通信网络。CAN 总线是一种在汽车上广泛采用的总线协议，被设计作为汽车环境中的微控制器通讯。
如果想了解更多的内容可以参考[CAN应用报告](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)。

## CAN FD 简介
CAN FD 是 CAN with Flexible Data rate 的缩写。也可以简单的认为是传统 CAN 的升级版。    
对比传统 CAN 总线技术，CAN FD 有两方面的升级：
* CAN FD 采用可变速率，最高速率可达 10Mb/s，而传统的CAN协议只有 1Mb/s。
* 能够支持更高的负载，在单个数据框架内传送率可达64字节，避免了经常发生的数据分裂状况。 

## 硬件连接
CAN 模块之间接线：CAN_H 接 CAN_H，CAN_L 接 CAN_L。

## DTS 节点配置
**只有 RK3562J 芯片才有 CAN 功能**
* 公共配置 `arch/arm64/boot/dts/rockchip/rk3562j.dtsi`

```
/ {
    can0: can@ff600000 {
        compatible = "rockchip,rk3562-can";
        reg = <0x0 0xff600000 0x0 0x1000>;
        interrupts = <GIC_SPI 64 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN0>, <&cru PCLK_CAN0>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_CAN0>, <&cru SRST_P_CAN0>;
        reset-names = "can", "can-apb";
        status = "disabled";
    };

    can1: can@ff610000 {
        compatible = "rockchip,rk3562-can";
        reg = <0x0 0xff610000 0x0 0x1000>;
        interrupts = <GIC_SPI 65 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&cru CLK_CAN1>, <&cru PCLK_CAN1>;
        clock-names = "baudclk", "apb_pclk";
        resets = <&cru SRST_CAN1>, <&cru SRST_P_CAN1>;
        reset-names = "can", "can-apb";
        status = "disabled";
    };
};
```

* 板级配置 `arch/arm64/boot/dts/rockchip/rk3562-firefly-aio-3562jq.dtsi`
```
&can0 {
    status = "okay";
    assigned-clocks = <&cru CLK_CAN0>;
    assigned-clock-rates = <200000000>;
    pinctrl-names = "default";
    pinctrl-0 = <&can0m1_pins>;
};
```

## 通信测试

### CAN FD 通信测试
使用 candump 和 cansend 工具进行收发报文测试即可。安装工具：
```bash
apt update
apt install -y can-utils
```

开始测试
```bash
#在收发端关闭can0设备
ip link set can0 down

#在收发端设置仲裁段1M波特率,数据段3M波特率                                          
ip link set can0 type can bitrate 1000000 dbitrate 3000000 fd on

#打印can0信息
ip -details link show can0

#在收发端打开can0设备
ip link set can0 up

#在接收端执行candump,阻塞等待报文                          	                 
candump can0

#canfd格式,在发送端执行cansend,发送报文
cansend can0 123##112345678

#can格式,在发送端执行cansend,发送报文                                     
cansend can0 123#1122334455667788                          
```

### CAN 通信测试    
同样使用 candump 和 cansend 工具进行收发报文测试。
```bash
#在收发端关闭can0设备
ip link set can0 down

#在收发端设置比特率为250Kbps                 
ip link set can0 type can bitrate 250000

#在收发端打开can0设备  	
ip link set can0 up

#在接收端执行candump,阻塞等待报文                        	
candump can0

#在发送端执行cansend，发送报文        	
cansend can0 123#1122334455667788  	
```

### 更多指令
```
1、 ip link set canX down 		//关闭can设备；
2、 ip link set canX up   		//开启can设备；
3、 ip -details link show canX 		//显示can设备详细信息；
4、 candump canX  			//接收can总线发来数据；
5、 ifconfig canX down 			//关闭can设备，以便配置;
6、 ip link set canX up type can bitrate 250000 //设置can波特率
7、 conconfig canX bitrate + 波特率；
8、 canconfig canX start 		//启动can设备；
9、 canconfig canX ctrlmode loopback on //回环测试；
10、canconfig canX restart 		// 重启can设备；
11、canconfig canX stop 		//停止can设备；
12、canecho canX 			//查看can设备总线状态；
13、cansend canX --identifier=ID+数据 	//发送数据；
14、candump canX --filter=ID：mask	//使用滤波器接收ID匹配的数据
```

## FAQS
总结调试过程中遇到的几个问题及解决方法：

### 报文发送后很久才接收到，或者接收不到。

检查总线 CAN_H 和 CAN_L， 杜邦线是否松动或者接反。

### CAN 时钟频率配置
如果 CAN 的比特率高于 1M 建议修改 CAN 时钟到 300M, 信号更稳定。低于 1M 比特率的, 时钟设置 200M 就可以。

修改 dts 配置中 can 的 assigned-clock-rates 属性即可修改时钟。
```
assigned-clock-rates = <200000000>;
```

**注意**：    
* 在某些时钟频率下，CAN的bitrate无法获得准确的速率，大家可以自行调整`assigned-clock-rates`去解决。
* 查看是否得到所需的bitrare：

```shell
ip -d link show can1
```

