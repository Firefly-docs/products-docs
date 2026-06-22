```
# The current directory is still update/, with package-file and all files listed in package-file.
# Copy the parameter file to paramter, because it is used by default by afptool.
$ cp Image/parameter.txt parameter
$ afptool -pack . ../update_new.img
------ PACKAGE ------
Add file: ./package-file
Add file: ./Image/MiniLoaderAll.bin
Add file: ./Image/parameter.txt
Add file: ./Image/trust.img
Add file: ./Image/uboot.img
Add file: ./Image/misc.img
Add file: ./Image/resource.img
Add file: ./Image/kernel.img
Add file: ./Image/boot.img
Add file: ./Image/recovery.img
Add file: ./Image/system.img
Add file: ./RESERVED
Add CRC...
------ OK ------
Pack OK!
```



