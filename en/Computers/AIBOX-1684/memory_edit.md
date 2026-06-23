# Memory allocation settings

A portion of the memory of AIBOX-1684 is allocated to NPU, VPP, and VPU devices. So the memory data obtained using commands such as `free -h` is inconsistent with the actual memory of AIBOX-1684.

If you need to view detailed memory allocation settings or if the default memory allocation conflicts with actual requirements, you can adjust the memory allocation by referring to the following documents.

## Install the memory allocation tool

Obtain the memory allocation tool package from [Download Center] and transfer it to AIBOX-1684. Run the following command to decompress the package:

```bash
tar -xvf memory_edit_v2.9.tar.xz
```

## View memory allocation settings

After decompressing the package, run the following command to view memory allocation settings:

```bash
cd memory_edit
./memory_edit.sh -p
```

The following is an example of memory allocation settings. The result displayed under `Info: get max memory size...` is the maximum memory that can be configured for each device, and the result displayed under `Info: get now memory size...` is the memory currently allocated for each device.

```bash
linaro@aibox-1684x:~/memory_edit$ ./memory_edit.sh -p
INFO: version: 2.9
Info: use dts file /home/linaro/memory_edit/output/bm1684x_se7_v1_mini.dts
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

## Modify memory allocation settings

You can set memory allocation by referring to the following command. The three parameters are decimal digits (unit: MiB) of the size to be configured for NPU, VPU, and VPP. The value can also be a hexadecimal value, in Byte.

```bash
# decimal, in MiB
./memory_edit.sh -c -npu 2048 -vpu 2048 -vpp 2048
# hexadecimal, in bytes
./memory_edit.sh -c -npu 0x80000000 -vpu 0x80000000 -vpp 0x80000000
```

After executing one of the above commands, check whether there is an Error in the output and whether the size of the three parts in the output similar to the following is the same as the size that needs to be configured.

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

If the check is correct, save the current work, replace the boot image in the boot partition with the modified `emmcboot.itb` file, and restart the machine for the changes to take effect.

```bash
sudo cp emmcboot.itb /boot
sync
sudo reboot
```

[Download Center]: https://en.t-firefly.com/doc/download/290.html