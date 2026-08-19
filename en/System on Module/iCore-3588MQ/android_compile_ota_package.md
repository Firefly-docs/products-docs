## OTA Compilation 

### Firefly full OTA compilation and upgrade
#### Add the -o parameter for OTA full compilation
```
./FFTools/make.sh -d rk3588-firefly-aio-3588q -j8 -l rk3588_firefly_aio_3588q-userdebug -o
```

#### Generate the full package path
```
out/target/product/rk3588-firefly-aio-3588q/rk3588-firefly-aio-3588q-ota-eng.xxx.zip 
```
#### Generate the base material package path
```
out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip
```
Copy the base material package and rename it, mainly for subcontracting upgrades.
```
cp out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip ~/ota/v1.zip 
```

#### Package compiled firmware and upgrades
```
./FFTools/mkupdate/mkupdate.sh -l rk3588_firefly_aio_3588q-userdebug
```
In the `rockdev/Image-rk3588-firefly-aio-3588q/` directory, find the corresponding firmware and burn it to the machine, then the machine can carry out the full package upgrade and the difference package upgrade,put 'update.zip' into /sdcard/ and restart the full package as prompted.

### Generate the incremental OTA package
#### Generate a new base material package 
The v1.zip basic material package has been made, and another v2.zip material package needs to be made to generate the difference package. You need to modify the kernel DTS code or Android code, and then run the command again.

```
./FFTools/make.sh -d rk3588-firefly-aio-3588q -j8 -l rk3588_firefly_aio_3588q-userdebug -o
```

Regenerate a new base material package that we can copy.
```
 cp out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip  ~/ota/v2.zip
```
#### Generate the incremental OTA package command
```
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```
#### Upgrade OTA incremental OTA package
Put update.zip to the machine sdcard directory reboot, it will automatically recognize, and then click the steps to complete the upgrade.

### OTA server setup

Refer to [OTA Documentation](https://en.t-firefly.com/doc/download/162.html#other_549) to setup the sever.