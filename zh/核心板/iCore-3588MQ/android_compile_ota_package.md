## OTA 编译 

### Firefly 完整OTA编译和升级
#### 添加-o 参数进行OTA全局编译
```
./FFTools/make.sh -d rk3588-firefly-aio-3588q -j8 -l rk3588_firefly_aio_3588q-userdebug -o
```

#### 生成完整包路径
```
out/target/product/rk3588-firefly-aio-3588q/rk3588-firefly-aio-3588q-ota-eng.xxx.zip 
```
#### 生成基础素材包路径
```
out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip
```
拷贝基础素材包并重新命名，主要用在差分包升级。
```
cp out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip ~/ota/v1.zip 
```

#### 打包编译的固件和升级
```
./FFTools/mkupdate/mkupdate.sh -l rk3588_firefly_aio_3588q-userdebug
```

在`rockdev/Image-rk3588-firefly-aio-3588q/` 目录下，找到对应的固件烧录到机器，此后机器就可以进行全量包升级和差分包升级，
把刚刚编译的完整包命名`rk3588-firefly-aio-3588q-ota-eng.xxx.zip` 为update.zip 放到/sdcard/目录下重启,并按照提示进行全量包升级。

### OTA 差分包生成
#### 生成新的基础素材包 
如上已经制作了v1.zip的基础素材包，需要再做一个v2.zip的素材包才能生成差分包，需修改内核dts相关代码或android的代码，然后重新执行命令
```
./FFTools/make.sh -d rk3588-firefly-aio-3588q -j8 -l rk3588_firefly_aio_3588q-userdebug -o
```

重新生成新的基础素材包我们可以进行拷贝

```
 cp out/target/product/rk3588-firefly-aio-3588q/obj/PACKAGING/target_files_intermediates/rk3588-firefly-aio-3588q-target_files-eng.xxx.zip  ~/ota/v2.zip
```
#### 生成差分包命令
```
out/host/linux-x86/bin/ota_from_target_files --block -v -i ~/ota/v1.zip -p out/host/linux-x86/ -k build/target/product/security/testkey ~/ota/v2.zip update.zip
```
#### 升级OTA差分包 
把update.zip 放到机器sdcard目录下重启，就会自动识别到，然后按步骤点击升级即可完成。

### OTA 服务器搭建

参考[OTA 文档](https://www.t-firefly.com/doc/download/196.html#other_638)搭建服务器。