## CAN 使用
### 简介
CAN(Controller Area Network)总线，即控制器局域网总线，是一种有效支持分布式控制或实时控制的串行通信网络。CAN总线是一种在汽车上广泛采用的总线协议，被设计作为汽车环境中的微控制器通讯。
如果想了解更多的内容可以参考[CAN应用报告](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)

### 硬件连接
CAN模块之间接线，CAN_H接CAN_H，CAN_L接CAN_L,

硬件原理图默认NC了CAN功能。
![](../../../rk3399_img/can3.png)

**由于默认硬件接口优先RS485，需要硬件对下方电阻进行修改**
![](../../../rk3399_img/can2.jpg)


### 通信测试
CAN通信测试，**驱动在AIO-3399ProC上默认已经做了支持**，所以直接使用 candump 和 cansend 工具进行收发报文测试即可，将工具push到/system/bin/目录下执行。工具可以在 [官方](http://www.t-firefly.com/share/index/index/id/3cacb04c663f9fe97bf494ca55763dcd.html) 或者 [github](https://github.com/linux-can/can-utils) 下载。
```
ip link set can0 type can bitrate 250000    	//在收发端设置比特率为250Kbps
ip link set can0 up                            	//在收发端打开can0设备
candump can0                                	//在接收端执行candump,阻塞等待报文
cansend can0 123#1122334455667788             	//在发送端执行cansend，发送报文
```
报文收发成功现象(这里分别用2台AIO-3399ProC做数据传输):
![](../../../rk3399_img/can1.png)
至此，MCP2515模块通信调试已经成功。

在测试的过程中发现，当设置比特率为500Kbps及以上时,如果使用 cansend 的时间间隔过短（使用频率太快），在接收端会出现多收、收错的现象。

### 更多指令
```
1、#ip link set canX down 		//关闭can设备；
2、#ip link set canX up   		//开启can设备；
3、#ip -details link show canX 		//显示can设备详细信息；
4、#candump canX  			//接收can总线发来数据；
5、#ifconfig canX down 			//关闭can设备，以便配置;
6、#ip link set canX up type can bitrate 250000 //设置can波特率
7、#conconfig canX bitrate + 波特率；
8、#canconfig canX start 		//启动can设备；
9、#canconfig canX ctrlmode loopback on //回环测试；
10、#canconfig canX restart 		// 重启can设备；
11、#canconfig canX stop 		//停止can设备；
12、#canecho canX 			//查看can设备总线状态；
13、#cansend canX --identifier=ID+数据 	//发送数据；
14、#candump canX --filter=ID：mask	//使用滤波器接收ID匹配的数据
```

### FAQS
总结调试过程中遇到的几个问题及解决方法：

Q: “ifconfig -a” 查看不到canX设备。

A: 检查驱动是否移植成功进入probe，检查MCP2515_CAN模块供电是否正常，是否损坏。

Q: 报文发送后很久才接收到，或者接收不到。

A：检查总线 CAN_H 和 CAN_L， 杜邦线是否松动或者接反。

Q:接收端只成功接收一次报文，后面就再也接收不到报文了。

A:检查MCP2515模块的INT中断引脚在dts中是否配对;INT引脚是否和开发板对应的pin脚连接上。

