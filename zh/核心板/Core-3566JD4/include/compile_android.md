* 单独编译 kernel：

```
cd ~/proj/Core-3566JD4/
./FFTools/make.sh -k -j8
```

* 单独编译 uboot：

```
cd ~/proj/Core-3566JD4/
./FFTools/make.sh -u -j8
```

* 单独编译 Android 上层：

```
cd ~/proj/Core-3566JD4/
./FFTools/make.sh -a -j8
```

* 同时编译 ubooot、kernel、Android：

```
cd ~/proj/Core-3566JD4/
./FFTools/make.sh -j8
```