# Device ID
## View Device ID

If you need to check the device ID, you can read the serial number of the core board. After reading successfully, it will return a string in json format:

```shell
linaro@bm1684:~$ cat /sys/bus/i2c/devices/1-0017/information
{
	"model": "SA5",
	"chip": "BM1684",
	"mcu": "STM32",
	"product sn": "HQDZKETBWY2100012",
	"board type": "0x01",
	"mcu version": "0x38",
	"pcb version": "0x12",
	"reset count": 0
}
```

## Update Device ID

The device ID update is generally used to update the product SN number, which is stored in the EEPROM of the MCU.

The user needs to modify it in the following ways:

(1) First you need to unlock the MCU EEPROM:

```shell
sudo -i
echo 0 > /sys/devices/platform/5001c000.i2c/i2c-1/1-0017/lock
```

(2) Write SN number:

```shell
echo "HQATEVBAIAIAI0001" > sn.txt
dd if=sn.txt of=/sys/bus/nvmem/devices/1-006a0/nvmem count=17 bs=1
```

(3) Finally, re-lock the MCU EEPROM to avoid accidental rewriting:

```shell
echo 1 > /sys/devices/platform/5001c000.i2c/i2c-1/1-0017/lock
```