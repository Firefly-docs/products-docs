* 单独编译 kernel：

```
cd ~/proj/IHC-3308GW/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/IHC-3308GW/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/IHC-3308GW/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/IHC-3308GW/
./FFTools/make.sh -j8
```