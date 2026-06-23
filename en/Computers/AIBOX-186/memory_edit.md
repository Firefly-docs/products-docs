# Memory Allocation Adjustment

A portion of the memory in AIBOX-186 is allocated to NPU, VPP, and VPU devices, so the memory data obtained using commands like `free -h` may not match the actual memory of the machine.

To view detailed memory allocation settings, or if the default memory allocation conflicts with actual needs and adjustments are required, please refer to the following documentation for the corresponding operations.

## Install Memory Allocation Tool

Obtain the memory allocation tool compressed package from the [download center] and transfer it to AIBOX-186. Execute the following command to unpack the compressed package:

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

## View Memory Allocation Settings

After unpacking the compressed package, execute the following command to view the memory allocation settings:

```bash
cd memory_edit
./memory_edit.sh -p
```

An example of the memory allocation settings result is shown below, where `Info: get max memory size ...` displays the maximum configurable memory for each device, and `Info: get now memory size ...` shows the currently allocated memory for each device.

```bash
linaro@aibox-1688x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1688x_se7_v1_mini.dts
Info: =======================================================================
Info: get ddr information ...
Info: ddr12_size 8589934592 Byte [8192 MiB]
Info: ddr3_size 4294967296 Byte [4096 MiB]
Info: ddr4_size 4294967296 Byte [4096 MiB]
Info: ddr_size 16384 MiB
Info: =======================================================================
Info: get max memory size ...
Info: max npu size: 0x1dbf00000 [7615 MiB]
Info: max vpu size: 0xc0000000 [3072 MiB]
Info: max vpp size: 0x100000000 [4096 MiB]
Info: =======================================================================
Info: get now memory size ...
Info: now npu size: 0xf6e00000 [3950 MiB]
Info: now vpu size: 0x80000000 [2048 MiB]
Info: now vpp size: 0xc0000000 [3072 MiB]
```

## Modify Memory Allocation Settings

You can refer to the following commands for memory allocation settings, where the three input parameters are decimal numbers representing the sizes needed for NPU, VPU, and VPP configurations in MiB; or hexadecimal values in Bytes.

```bash
# Decimal, unit MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# Hexadecimal, unit Byte
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

After executing one of the above commands, check the output for any errors, and verify whether the sizes of the three sections in the output are the same as the required configurations.

```bash
Info: output configuration results ...
Info: vpu mem area(ddr3): 0x8000000 [128 MiB] 0x20000000 -> 0x27ffffff
Info: ion npu mem area(ddr1): 0x80000000 [2048 MiB] 0x24100000 -> 0xa40fffff
Info: ion vpu mem area(ddr3): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: ion vpp mem area(ddr4): 0x80000000 [2048 MiB] 0x80000000 -> 0xffffffff
Info: =======================================================================
Info: start check memory size ...
Info: check npu size: 0x80000000 [2048 MiB]
Info: check vpu size: 0x80000000 [2048 MiB]
Info: check vpp size: 0x80000000 [2048 MiB]
Info: check edit size ok
Info: en_emmcfile ok
```

If the checks are correct, please save your current work and follow the steps below to replace the modified emmcboot.itb file in the boot partition with the boot image, and finally restart the machine to apply the changes.

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

[Download Center]: https://www.t-firefly.com/doc/download/266.html