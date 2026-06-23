# 设备 ID
## 查看设备 ID

需要查看设备 ID 可通过读取核心板序列号，读取成功后会返回 json 格式的字符串：

```shell
linaro@bm1688:~$ cat /sys/bus/i2c/devices/1-0017/information
{
	"model": "SA5",
	"chip": "RK3576",
	"mcu": "STM32",
	"product sn": "HQDZKETBWY2100012",
	"board type": "0x01",
	"mcu version": "0x38",
	"pcb version": "0x12",
	"reset count": 0
}
```

## 更新设备 ID

设备 ID 更新一般用于更新产品 SN 号，它存放在 MCU 的 EEPROM 中。

用户需要修改，可以通过如下方式：

（1）首先需要解锁 MCU EEPROM：

```shell
sudo -i
echo 0 > /sys/devices/platform/5001c000.i2c/i2c-1/1-0017/lock
```

（2）写入 SN 号：

```shell
echo "HQATEVBAIAIAI0001" > sn.txt
dd if=sn.txt of=/sys/bus/nvmem/devices/1-006a0/nvmem count=17 bs=1
```

（3）最后重新对 MCU EEPROM 加锁，以避免意外改写：

```shell
echo 1 > /sys/devices/platform/5001c000.i2c/i2c-1/1-0017/lock
```