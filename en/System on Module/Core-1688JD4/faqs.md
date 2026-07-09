# FAQs

## SDK Release
The SDK available for user development is being organized and is expected to be released in a week.

## What are the default username and password for the system?  
* Username: `linaro`
* Password: `linaro`
* Switch to superuser: `sudo -s`

## What should I do if the system starts abnormally and keeps rebooting?

It may be due to insufficient power current; please use a power supply with a voltage of 12V and a current of 5A or more.


## Memory
* System Memory : `free -h`
* ION Memory
    * NPU : `cat /sys/kernel/debug/ion/cvi_npu_heap_dump/summary`
    * VPP : `cat /sys/kernel/debug/ion/cvi_vpp_heap_dump/summary`

