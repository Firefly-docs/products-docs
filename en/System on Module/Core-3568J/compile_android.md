* Compile the kernel separately：

```
cd ~/proj/Core-3568J/
./FFTools/make.sh -k -j8
```

* Compile the uboot separately：

```
cd ~/proj/Core-3568J/
./FFTools/make.sh -u -j8
```

* Compile Android upper layer separately：

```
cd ~/proj/Core-3568J/
./FFTools/make.sh -a -j8
```

* Compile ubooot, kernel, Android at the same time：

```
cd ~/proj/Core-3568J/
./FFTools/make.sh -j8
```