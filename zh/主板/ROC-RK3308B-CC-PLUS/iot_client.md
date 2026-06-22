# Modbus转Mqtt网关技术案例

本网关提供了程序`iot_client`，为用户提供便捷的数据采集、数据上云的功能，设备端支持`ModbusRtu`、`ModbusTcp`工业协议，云端支持EMQX、阿里云。

## 参数配置

`iot_client`的所有配置均通过`JSON`配置文件来实现，所有配置文件存放在`/usr/share/iot_client`目录中。

### 1 全局配置

全局配置用于设置网关向上连接的云服务器，和向下连接的连接器。

全局配置文件的路径为：`/usr/share/iot_client/general_config.json`，例子如下：

```json
{
	"cloud": {
		"type": "emqx",
		"configuration": "emqx.json"
	},
	"connectors": [{
		"name": "modbus_serial_ttysWK0",
		"type": "modbus",
		"configuration": "modbus_serial_ttysWK0.json"
	}]
}
```

#### 1.1 子节点cloud

该节点描述云端的类型和云端的配置文件，目前只支持阿里云和EMQX。

| 参数          | 默认值 | 描述和说明                                          |
| ------------- | ------ | --------------------------------------------------- |
| type          |        | 云服务器类型，取值"emqx"、"aliyun"                  |
| configuration |        | 云服务器配置文件，存放在`/usr/share/iot_client`目录 |

上面配置案例表明网关使用EMQX作为云服务器，配置文件为`emqx.json`。

#### 1.2 子节点connectors

该节点描述网关所连接的连接器，连接器可以是`modbus`连接器，`CAN`总线连接器等等，目前只支持`modbus`连接器，它是对协议操作方法的描述。一个网关可以连接0个或多个连接器，一个连接器上可以连接多个使用相同协议的外设。

| 参数          | 默认值 | 描述和说明                                                   |
| ------------- | ------ | ------------------------------------------------------------ |
| name          |        | 连接器名称，字段不要有空格，由字母、数字、下划线组成，且必须唯一 |
| type          |        | 连接器类型，目前只能取值"modbus"                             |
| configuration |        | 连接器配置文件，存放在`/usr/share/iot_client`目录            |

上面配置案例表明网关连接了一个名为`modbus_serial_ttysWK0`的`modbus`连接器，该连接器的配置文件为`modbus_serial_ttysWK0.json`。

##### 1.2.1 添加多个连接器

`connectors`子节点是一个`json`数组，可以添加多个连接器，eg：

```json
{
	"cloud": {
		"type": "emqx",
		"configuration": "emqx.json"
	},
	"connectors": [{
		"name": "modbus_serial_ttysWK0",
		"type": "modbus",
		"configuration": "modbus_serial_ttysWK0.json"
	},{
		"name": "modbus_serial_ttysWK1",
		"type": "modbus",
		"configuration": "modbus_serial_ttysWK1.json"
	}]
}
```

##### 1.2.2 不添加任何连接器

如果不连接任何外设，直接取空即可。

```json
{
	"cloud": {
		"type": "emqx",
		"configuration": "emqx.json"
	},
	"connectors": []
}
```

### 2 云配置

网关只能连接一个云平台，只能设置连接EMQX或者阿里云平台。

#### 2.1 EMQX云配置

例子：`[/usr/share/iot_client/emqx.json]`

```json
{
	"host": "127.0.0.1",
	"port": 8883,
	"ssl": true,
	"qos": 1,
	"keepAlive": 60,
	"username": "firefly",
	"password": "",
	"clientId": ""
}
```

参数配置说明如下表格：

| 参数      | 缺省默认值           | 描述和说明                                 |
| --------- | -------------------- | ------------------------------------------ |
| host      |                      | 云服务器IP地址                             |
| port      |                      | 端口号（根据使用tcp还是ssl填写相应端口号） |
| ssl       | false                | 是否使用ssl连接                            |
| qos       | 1                    | qos=1设置mqtt至少发送一次                  |
| keepAlive | 60                   | 保活时间                                   |
| username  |                      | 用户名                                     |
| password  |                      | 密码                                       |
| clientId  | 由iot_client自动生成 | 客户端ID                                   |

注意：若使用`ssl`连接，需配置用户自己的`ca`证书文件。`ca`证书文件固定路径为：`/usr/share/iot_client/cacert.pem`，用户将文件重命名覆盖即可。

#### 2.2 阿里云配置

如果配置了EMQX云配置，请跳过当前章节。

例子1：阿里云一机一码配置：`[/usr/share/iot_client/aliyun_normal.json]`

```json
{
    "region": "cn-shanghai",
    "is_public_instance" : true,
    "is_dynreg" : false,
    "is_data_mode" : true,
    "product_key" : "a1lrT6i2bBF",
    "device_name" : "rk3308_1",
    "device_secret" : "8a313b03c4ad7c13fc4a3ed8304b7f7d"
}
```

例子2：阿里云一型一密(白名单)配置：`[/usr/share/iot_client/aliyun_whitelist.json]`

```json
{
	"region":	"cn-shanghai",
	"is_public_instance":	true,
	"is_dynreg":	true,
	"is_data_mode" : true,
	"product_key":	"a1lrT6i2bBF",
	"device_name":	"rk3308_5",
	"is_white_list":	true,
	"product_secret":	"KFgItEca9W14uCja"
}
```

例子3：阿里云一型一密(免白名单)配置：`[/usr/share/iot_client/aliyun_no_whitelist.json]`

```json
{
	"region":	"cn-shanghai",
	"is_public_instance":	true,
	"is_dynreg":	true,
	"is_data_mode" : true,
	"product_key":	"a1lrT6i2bBF",
	"device_name":	"rk3308_6",
	"is_white_list":	false,
	"product_secret":	"KFgItEca9W14uCja"
}
```

参数配置说明如下表格：

| 参数               | 缺省默认值 | 描述和说明                                          |
| ------------------ | ---------- | --------------------------------------------------- |
| region             |            | 区域                                                |
| is_public_instance |            | 是否是共用实例                                      |
| is_dynreg          |            | 是否是动态注册                                      |
| is_data_mode       | 缺省为true | 是否采用物模型                                      |
| product_key        |            | 产品key                                             |
| product_secret     |            | 产品秘钥，在is_dynreg为true时必须提供该字段         |
| device_name        |            | 设备名                                              |
| device_secret      |            | 设备秘钥，在is_dynreg为false时必须提供该字段        |
| is_white_list      |            | 是否是白名单模式，在is_dynreg为true时必须提供该字段 |

### 3 连接器配置

#### 3.1 modbus连接器配置

eg: `[/usr/share/iot_client/modbus_serial_ttysWK0.json]`（这是一个`ModbusRtu`协议的配置文件）

```json
{
	"name": "modbus_serial_ttysWK0",
	"type": "rtu",
	"port": "/dev/ttysWK0",
	"stopbits": 1,
	"bytesize": 8,
	"parity": "N",
	"baudrate": 4800,
	"devices": [{
		"unitId": 1,
		"deviceName": "RS_WS_N01",
		"uploadInterval": 1000,
		"cycleInterval": 1000,
		"sendDataOnlyOnChange": false,
		"order": "dcba",
		"attributes": [{
			"tag": "humidity",
			"type": "int16",
			"rw_access": "r",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"address": 40001
		}, {
			"tag": "temperature",
			"type": "int16",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"rw_access": "r",
			"address": 40002
		}]
	}, {
		"unitId": 2,
		"deviceName": "RS_WS_N02",
		"uploadInterval": 1000,
		"cycleInterval": 1000,
		"sendDataOnlyOnChange": false,
		"order": 18,
		"attributes": [{
			"tag": "humidity",
			"type": "int16",
			"rw_access": "r",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"address": 40001
		}, {
			"tag": "temperature",
			"type": "int16",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"rw_access": "r",
			"address": 40002
		}]
	}]
}
```

##### 3.1.1 ModbusRtu配置

一个最简单的`ModbusRtu`配置文件如下

```
{
	"name": "modbus_serial_ttysWK0",
	"type": "rtu",
	"port": "/dev/ttysWK0",
	"stopbits": 1,
	"bytesize": 8,
	"parity": "N",
	"baudrate": 4800,
	"devices": []
}
```

| 参数     | 默认值 | 描述和说明                                                   |
| -------- | ------ | ------------------------------------------------------------ |
| name     |        | 连接器名称，字段不要有空格，由字母、数字、下划线组成         |
| type     |        | 类型，取值rtu/tcp                                            |
| port     |        | 串口设备文件                                                 |
| stopbits |        | 停止位，取值1,2                                              |
| bytesize |        | 数据位，取值5,6,7,8                                          |
| parity   |        | 奇偶校验位，N：无校验，E：偶校验，O：奇校验                  |
| baudrate |        | 波特率，取值300,1200,2400,4800,9600,19200,38400,56000,115200,187500,230400 |
| devices  |        | 参考3.1.3节                                                  |

##### 3.1.2 ModbusTcp配置

一个最简单的`ModbusTcp`配置文件如下，

```
{
	"name": "modbus_tcp",
	"type": "tcp",
	"host": "168.168.101.95",
	"port": 502,
	"devices": []
}
```

| 参数    | 默认值 | 描述和说明                                           |
| ------- | ------ | ---------------------------------------------------- |
| name    |        | 连接器名称，字段不要有空格，由字母、数字、下划线组成 |
| type    |        | 类型，取值rtu/tcp                                    |
| host    |        | IP地址                                               |
| port    |        | 端口号                                               |
| devices |        | 参考3.1.3节                                          |

##### 3.1.3 节点devices：设备配置

设备配置包含以下信息：

```
{
			"unitId":	1,
			"deviceName":	"Test Device A1",
			"uploadInterval":	1000,
			"cycleInterval":	1000,
			"sendDataOnlyOnChange":	true,
			"order":	"dcba",
			"attributes":	[]
}	
```

| 参数                 | 默认值 | 描述和说明                               |
| -------------------- | ------ | ---------------------------------------- |
| unitId               |        | slave id                                 |
| deviceName           |        | 设备名，同一连接器的不同设备名，必须唯一 |
| uploadInterval       |        | 上传间隔，单位ms                         |
| cycleInterval        |        | 轮循间隔，单位ms                         |
| sendDataOnlyOnChange |        | 是否只在数据发生改变才提交数据           |
| order                |        | 字节序,取值"abcd","cdab","dcba","badc"   |
| attributes           |        | 参考3.1.4节                              |

这里补充一下`order`属性

在`modbus`中，除了操作线圈寄存器和离散寄存器的大小是以1个字节为单位外，其他寄存器（保持寄存器，输入寄存器）每次操作的单位都是2个字节。而外设的体系结构不同，导致数据有不同的大小端字节序。

大端：高字节存放在低地址，低字节存放在高地址，比如：

```
char buf[4] = 0x12345678;（左边是数据的高位）
低地址 -----------------> 高地址
0x12  |  0x34  |  0x56  |  0x78
```

小端：高字节存放在高地址，低字节存放在低地址，比如：

```
char buf[4] = 0x12345678;（左边是数据的高位）
低地址 ------------------> 高地址
0x78  |  0x56  |  0x34  |  0x12
```

如果一个属性是由2个属性构成的，2个寄存器保存的值哪个在前， 就需要设置寄存器高低位在前。只由一个寄存器保存的变量是不需要设置寄存器高低位在前的。比如：

```
有一个属性，由2个连续寄存器组成，在内存中某一个时刻其数据为：0x1234（左边是数据的高位）
高位在前时，读取出来为：12  34
低位在前时，读取出来为：34  12
```

根据大小端，和高低位，数据在`modbus`的存储格式就有4种：`"abcd","cdab","dcba","badc"`。

```
"abcd"：大端模式 + 高位在前
"cdab"：大端模式 + 低位在前
"badc"：小端模式 + 高位在前
"dcba"：小端模式 + 低位在前
```

##### 3.1.4 节点attributes：属性配置

一个完整的属性配置为：

```
{
        "tag":	"bit_coil_rw", 						
        "type":	"bit",								
        "rw_access":	"rw",							
        "address":	4,								
        "variableRangeConversion": 1,	              
        "bitDigit": 0,									
        "doubleDigit": 0						
}
```

| 参数                    | 默认值      | 描述和说明                                                   |
| ----------------------- | ----------- | ------------------------------------------------------------ |
| tag                     |             | 属性名称，字段不要有空格，由字母、数字、下划线组成。若是同一设备的不同属性名，必须唯一 |
| type                    |             | 属性类型，取值bool,bit,int16,uint16,int32,uint32,int64,uint64,float,double |
| rw_access               |             | 读写权限，rw(可读可写)，r(只读)，w(只写)                     |
| address                 |             | PLC地址                                                      |
| variableRangeConversion | 省略默认值1 | 量程转换，取值1，(0.1)^N，(10)^N  （N为正整数）              |
| bitDigit                | 省略默认值0 | 寄存器的第几位，取值0-15                                     |
| doubleDigit             | 省略默认值0 | 浮点数小数点保留位数                                         |

**补充：**

前面介绍过，`modbus`寄存器分为线圈寄存器，离散寄存器，输入寄存器和保持寄存器。

①其中，离散寄存器和输入寄存器是只读的，线圈寄存器和保持寄存器可读可写的。

②不同的寄存器类型，人为的将`PLC`地址规定为不同的地址类型

| 地址类型 | 寄存器类型     | PLC地址范围               |
| -------- | -------------- | ------------------------- |
| 0X       | 线圈寄存器地址 | 1~10000                   |
| 1X       | 离散寄存器地址 | 10001~20000,110001~165535 |
| 3X       | 输入寄存器地址 | 30001~40000,310001~365535 |
| 4X       | 保持寄存器地址 | 40001~50000,410001~465535 |

**type**：属性类型

目前支持的属性类型为`bool,bit,int16,uint16,int32,uint32,int64,uint64,float,double`

对于线圈寄存器或离散寄存器，只支持`bool`和`bit`类型

对于输入寄存器或保持寄存器，支持所有类型

**rw_access**：读写

对于不同的寄存器类型，能够选择的`rw_access`就会有所区别，比如线圈寄存器和保持寄存器可取值为`"r","w","rw"`，离散寄存器和保持寄存器只能取值`"r"`

**address**：`PLC`地址

`modbus`实际上并非使用`PLC`地址作为读写地址，而是使用`modbus`地址。由于`modbus`地址是从0开始的，其换算规则如下：

| PLC地址范围               | modbus地址换算                 |
| ------------------------- | ------------------------------ |
| 1~10000                   | =PLC地址-1                     |
| 10001~20000,110001~165535 | =PLC地址-10001 =PLC地址-110001 |
| 30001~40000,310001~365535 | =PLC地址-30001 =PLC地址-310001 |
| 40001~50000,410001~465535 | =PLC地址-40001 =PLC地址-410001 |

比如某个属性的地址为`411234`，则实际上`modbus`读写的地址为`411234-410001=1233`。

**variableRangeConversion**：量程转换

某些从modbus读取到的数值并非实际数值，需要经过数值的缩小或放大，比如某PLC手册：

```
寄存器地址 内容 操作
40001H 湿度(单位 0.1%RH) 只读
40002H 温度(单位 0.1℃) 只读

温度：
当温度低于 0℃时以补码形式上传
FF9B H(十六进制)=-101=>温度=-10.1℃
湿度：
292 H(十六进制)=658=>湿度=65.8%RH
```

可见，数值需要输小10倍，才是真实的数值，`variableRangeConversion`此时取值为0.1

**doubleDigit**：小数点保留位数

同样以上面为例，`doubleDigit`需要取值为1，小数点后保留1位有效数字

**bitDigit**：寄存器的第几位

如果某个属性的数值，来源于某个寄存器的第几位，就需要设置这个参数了。注意类型为bool或bit，这个字段才有用。eg：

```
{
        "tag":	"rw_bool",
        "type":	"bool",
        "rw_access":	"rw",
        "bitDigit":	13,
        "address":	40004
}
```

## 案例：使用阿里云服务器

本案例网关使用阿里云作为云服务器，外设使用温湿度传感器，该温湿度传感器使用标准`ModbusRtu`协议。

- 登录阿里云平台，控制台->物联网平台

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/into_aliyun_1.png)

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/into_aliyun_2.png)

- 公共实例->产品->创建产品

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/into_aliyun_3.png)

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/start.png)

- 新建产品，输入自定义产品名称，设置为网关设备，联网采用蜂窝网络

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/add_product.png)

- 配置产品功能

点击刚刚创建的产品

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_1.png)

点击：功能定义->编辑草稿

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_2.png)

点击：添加模块

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_3.png)

添加模块：创建温湿度传感器1

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_4.png)

添加模块：创建温湿度传感器2

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_5.png)

点击：温湿度传感器1->添加自定义功能

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/product_config_6.png)

添加属性：温度检测

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/module_config_1.png)

添加属性：湿度检测

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/module_config_2.png)

同理，给温度传感器2添加同样的属性，这里略

最后，发布上线即可。

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/module_config_3.png)

- 新建设备

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/add_device_1.png)

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/add_device_2.png)

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/add_device_3.png)

- 复制产品ProductKey，设备名DeviceName，设备秘钥DeviceSecret，后面网关需要用到。

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/set_device_config.png)

- 使用USB TYPE C线接入网关，另一端接入电脑

网关所有的配置文件都在/usr/share/iot_client目录下，进入/usr/share/iot_client

```bash
cd /usr/share/iot_client
```

编辑网关全局配置文件`general_config.json`

```bash
vi general_config.json
```

更改文件的内容如下：

```json
{
	"cloud": {
		"type": "aliyun",
		"configuration": "aliyun_normal.json"
	},
	"connectors": [{
		"name": "modbus_serial",
		"type": "modbus",
		"configuration": "modbus_serial.json"
	}]
}
```

上述文件的意义:

①配置云服务器类型为`aliyun`，配置文件为`aliyun_normal.json`

②配置连接器类型为`modbus`，配置文件为`modbus_serial.json`

- 配置云服务器文件`aliyun_normal.json`

配置网关以非动态注册的方式连接阿里云，以物模型数据将网关的数据采集上报

```bash
vi aliyun_normal.json
```

更改product_key，device_name，device_secret的值为上面复制的值，其他保留不变。

```json
{
    "region": "cn-shanghai",
    "is_public_instance" : true,
    "is_dynreg" : false,
    "is_data_mode" : true,
    "product_key" : "你自己产品的ProductKey",
    "device_name" : "你自己设备的DeviceName",
    "device_secret" : "你自己设备的DeviceSecret"
}
```

比如，例子中的配置文件如下：

```json
{
    "region": "cn-shanghai",
    "is_public_instance" : true,
    "is_dynreg" : false,
    "is_data_mode" : true,
    "product_key" : "a17pmHuRtr4",
    "device_name" : "ihc-3308Y-device1",
    "device_secret" : "3d0ad657f020fe601d0c80a3f880832b"
}
```

- 配置连接器modbus配置文件`modbus_serial.json`

根据PLC连接的串口，以及PLC手册配置串口的基本信息

```json
{
	"name":	"modbus_serial",
	"type":	"rtu",
	"port":	"/dev/ttysWK0",
	"stopbits":	1,
	"bytesize":	8,
	"parity":	"N",
	"baudrate":	4800,
	"devices":	[]
}
```

由于我们连接了2个PLC在"/dev/ttysWK0"即RS485_1上，查看PLC手册的信息，将这个2个PLC的具体信息加入上面的"devices"，

```json
{
	"name":	"modbus_serial",
	"type":	"rtu",
	"port":	"/dev/ttysWK0",
	"stopbits":	1,
	"bytesize":	8,
	"parity":	"N",
	"baudrate":	4800,
	"devices":	[{
			"unitId":	1,
			"deviceName":	"DEV1",
			"uploadInterval":	1000,
			"cycleInterval":	1000,
			"sendDataOnlyOnChange":	true,
			"order":	"dcba",
			"attributes":	[{
					"tag":	"humidity",
					"type":	"int16",
					"rw_access":	"r",
					"variableRangeConversion":	0.1,
					"doubleDigit":	2,
					"address":	40001
				}, {
					"tag":	"temperature",
					"type":	"int16",
					"variableRangeConversion":	0.1,
					"doubleDigit":	2,
					"rw_access":	"r",
					"address":	40002
				}]
		}, {
			"unitId":	2,
			"deviceName":	"DEV2",
			"uploadInterval":	1000,
			"cycleInterval":	1000,
			"sendDataOnlyOnChange":	false,
			"order":	"dcba",
			"attributes":	[{
					"tag":	"humidity",
					"type":	"int16",
					"rw_access":	"r",
					"variableRangeConversion":	0.1,
					"doubleDigit":	2,
					"address":	40001
				}, {
					"tag":	"temperature",
					"type":	"int16",
					"variableRangeConversion":	0.1,
					"doubleDigit":	2,
					"rw_access":	"r",
					"address":	40002
				}]
		}]
}
```

之后，运行iot_client，将PLC采集到的数据发送到阿里云，打印日志如下：

```
~ # iot_client 
establish mbedtls connection with server(host='a17pmHuRtr4.iot-as-mqtt.cn-shanghai.aliyuncs.com', port=[443])
success to establish tcp, fd=4
success to establish mbedtls connection, fd = 4(cost 45110 bytes in total, max used 47786 bytes)
[INFO ] "2021-03-02 16:57:49" (iot_aliyun_core.c:60) - Aliyun connect success!
[INFO ] "2021-03-02 16:57:49" (iot_client.c:140) - Iot client started!
[INFO ] "2021-03-02 16:57:49" (iot_aliyun_core.c:632) - Subscribing to topic /a17pmHuRtr4/ihc-3308Y-device1/user/sys_request success!
[INFO ] "2021-03-02 16:57:49" (iot_aliyun_core.c:632) - Subscribing to topic /a17pmHuRtr4/ihc-3308Y-device1/user/data_request success!
[INFO ] "2021-03-02 16:57:49" (iot_aliyun_core.c:632) - Subscribing to topic /a17pmHuRtr4/ihc-3308Y-device1/user/cloud_response success!
Opening /dev/ttysWK0 at 4800 bauds (N, 8, 1)
[INFO ] "2021-03-02 16:57:50" (iot_modbus_core.c:577) - Connecting to modbus(type:rtu,slaveId:1) success!
[01][03][00][00][00][01][84][0A]
Waiting for a confirmation...
<01><03><02><00><DE><38><1C>
[01][03][00][01][00][01][D5][CA]
Waiting for a confirmation...
<01><03><02><00><64><B9><AF>
[INFO ] "2021-03-02 16:57:51" (iot_aliyun_core.c:395) - Publishing payload: {"DEV1:humidity":22.2,"DEV1:temperature":10}
...
```

进入阿里云，点击设备->物模型数据->温湿度传感器，即可查看到从本地上传的物模型数据。

![](../../../rk3308_img/ROC-RK3308B-CC-PLUS/iot_client/aliyun_data.png)

## 案例：使用EMQX服务器

本案例网关使用EMQX作为云服务器，外设使用温湿度传感器，该温湿度传感器使用标准`ModbusRtu`协议。

设置全局配置文件，设置云服务器类型为emqx，配置文件为`emqx.json`，连接器类型为`modbus`，配置文件为`modbus_serial_ttysWK0.json`

`vi /usr/share/iot_client/general_config.json`

```json
{
	"cloud": {
		"type": "emqx",
		"configuration": "emqx.json"
	},
	"connectors": [{
		"name": "modbus_serial_ttysWK0",
		"type": "modbus",
		"configuration": "modbus_serial_ttysWK0.json"
	}]
}
```



设置EMQX服务器信息`emqx.json`。由于使用tcp连接，ssl字段取值为false。

`vi /usr/share/iot_client/emqx.json`

```json
{
	"host":	"168.168.100.172",
	"port":	1883,
	"ssl": false,
	"qos":	1,
	"keepAlive":	60,
	"username":	"firefly",
	"password":	"",
	"clientId":	""
}
```



设置连接器`modbus_serial_ttysWK0.json`

`vi /usr/share/iot_client/modbus_serial_ttysWK0.json`

```json
{
	"name": "modbus_serial_ttysWK0",
	"type": "rtu",
	"port": "/dev/ttysWK0",
	"stopbits": 1,
	"bytesize": 8,
	"parity": "N",
	"baudrate": 4800,
	"devices": [{
		"unitId": 1,
		"deviceName": "RS_WS_N01",
		"uploadInterval": 1000,
		"cycleInterval": 1000,
		"sendDataOnlyOnChange": false,
		"order": "dcba",
		"attributes": [{
			"tag": "humidity",
			"type": "int16",
			"rw_access": "r",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"address": 40001
		}, {
			"tag": "temperature",
			"type": "int16",
			"variableRangeConversion": 0.1,
			"doubleDigit": 2,
			"rw_access": "r",
			"address": 40002
		}]
	}]
}
```



运行iot_client，打印日志如下：

```
/usr/share/iot_client # iot_client 
[INFO ] "2021-03-02 14:34:43" (iot_emqx_core.c:311) - ClientId:mqtt-gateway-5c07c88b92014d87925c73718fab7231
[INFO ] "2021-03-02 14:34:43" (iot_emqx_core.c:312) - Connect to tcp://168.168.100.172:1883 success!
[INFO ] "2021-03-02 14:34:43" (iot_client.c:140) - Iot client started!
[INFO ] "2021-03-02 14:34:43" (iot_emqx_core.c:372) - Subscribing to topic v1/gateway/mqtt-gateway-5c07c88b92014d87925c73718fab7231/sys_request success!
[INFO ] "2021-03-02 14:34:43" (iot_emqx_core.c:372) - Subscribing to topic v1/gateway/mqtt-gateway-5c07c88b92014d87925c73718fab7231/data_request success!
[INFO ] "2021-03-02 14:34:43" (iot_emqx_core.c:372) - Subscribing to topic v1/gateway/mqtt-gateway-5c07c88b92014d87925c73718fab7231/cloud_response success!
Opening /dev/ttysWK0 at 4800 bauds (N, 8, 1)
[INFO ] "2021-03-02 14:34:43" (iot_modbus_core.c:577) - Connecting to modbus(type:rtu,slaveId:1) success!
[01][03][00][00][00][01][84][0A]
Waiting for a confirmation...
<01><03><02><00><00><B8><44>
[01][03][00][01][00][01][D5][CA]
Waiting for a confirmation...
<01><03><02><00><00><B8><44>
[INFO ] "2021-03-02 14:34:44" (iot_emqx_core.c:355) - Publishing to topic v1/cloud/mqtt-gateway-5c07c88b92014d87925c73718fab7231 : {"request":"attribute_send","packet_id":1,"timestamp":1614666884,"code":0,"msg":"","data":{"device_name":"RS_WS_N01","attribute":{"humidity":0,"temperature":0}}}
```