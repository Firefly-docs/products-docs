# CAN

CAN(Controller Area Network)总线，即控制器局域网总线，是一种有效支持分布式控制或实时控制的串行通信网络。CAN总线是一种在汽车上广泛采用的总线协议，被设计作为汽车环境中的微控制器通讯。如果想了解更多的内容可以参考[CAN应用报告](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf)。

AIO-Orin Nano 的 CAN 接口（凤凰端子座）位置如下图所示：

<center>

<img alt="AIO-Orin Nano CAN 接口" src="../../../../nvidia_img/AIO-Orin-Nano/can_terminal_bare_board.png" width="400">

</center>

由于只有一个 CAN，所以默认在内核中，第一个创建的设备为 `can0`。

## CAN 通信测试 [step]

使用 candump 和 cansend 工具进行收发报文测试即可:

```
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

## FAQS

### 报文发送后很久才接收到，或者接收不到。

检查总线 CAN_H 和 CAN_L，杜邦线是否松动或者接反。